# Plan for Integrating `LangExtract` into Mnemosyne

This document outlines the architectural changes and implementation steps required to integrate Google's `LangExtract` library into the Mnemosyne project. The goal is to use `LangExtract` to generate an additional, automated ground truth and to enable experiments to be evaluated against multiple ground truth sources simultaneously.

### Part 1: Create a `LangExtract` Ground Truth Generator

The first step is to create a standalone utility that can process a source `.docx` file and generate a new ground truth `.json` file using `LangExtract`. This will be a one-time process for each source document.

#### **1.1. New Utility Class: `GroundTruthGenerator`**

A new class, `GroundTruthGenerator`, will be created. This class will encapsulate all the logic for using `LangExtract`.

  * **Dependencies**: It will require the `langextract` library to be installed (`pip install langextract`).
  * **Initialization**: It will be initialized with your Gemini API key.
  * **Core Method**: It will have a primary method, `generate_from_docx(source_filepath, output_filepath)`.

#### **1.2. Workflow**

The `generate_from_docx` method will perform the following steps:

1.  **Load Document**: Read the text content from the source `.docx` file.
2.  **Call `LangExtract`**: Pass the text to the `langextract.extract_entities_and_relations` function.
3.  **Transform Output**: The output from `LangExtract` will need to be transformed into the same JSON structure as your existing ground truth files (`nodes` and `relationships`). This involves creating unique IDs and separating entities into `:Instance` and `:Type` nodes.
4.  **Apply Normalization**: Run the new JSON data through the same `displayName` and lowercase `name` normalization script you created (`json_update_script`) to ensure it's consistent for matching.
5.  **Save File**: Save the final, formatted JSON to the specified output path.

This utility can be a separate cell in your notebook or a standalone Python script that you run as needed to pre-process your source documents.

### Part 2: Implement Multi-Ground Truth Evaluation

The core of your experiment runner needs to be updated to support comparing one generated graph against a list of different ground truth files.

#### **2.1. Update Experiment Definitions**

The `er_file_name` key in your `experiment_groups` configuration will be updated to accept either a single string (for backward compatibility) or a list of strings.

**Example:**

```
"er_file_name": [
    "NER Test 1 - ER_manual.json", 
    "NER Test 1 - ER_langextract.json"
],
```

#### **2.2. Add a Cleanup Method to `GraphDBRefiner`**

To ensure each ground truth comparison is clean, a new method will be added to `GraphDBRefiner` to delete only the golden standard nodes before loading the next one.

**New Method: `clear_golden_standard_data()`**

```
def clear_golden_standard_data(self):
    """Deletes all nodes and relationships labeled as the golden standard."""
    with self.driver.session() as session:
        # Uses the static GOLDEN_STANDARD_EXP_ID to find and delete the nodes
        query = f"""
            MATCH (n:`EXP_{Experiment.GOLDEN_STANDARD_EXP_ID}`)
            DETACH DELETE n
        """
        session.run(query)
    logging.info("Cleared previous Golden Standard data from Neo4j.")
```

#### **2.3. Modify the `Experiment` Class Logic**

The `_run_er_test` method in the `Experiment` class will be refactored to handle the list of ground truth files.

**Updated Workflow for `_run_er_test`:**

1.  **Check `er_file_name` Type**: The method will first check if `self.config.ER_FILE_PATH` is a list or a single path.
2.  **Loop Through Ground Truths**: If it's a list, it will loop through each file path.
3.  **Inside the Loop**: For each ground truth file, the loop will:
    a. Call `graph_db.refiner.clear_golden_standard_data()` to wipe the previous ground truth.
    b. Call `self._load_golden_standard()` with the current ground truth file path to load the new one.
    c. Run the comparison logic (`get_cypher_comparison_scores`).
    d. Store the resulting `TestResult` object in a list, making sure to include the name of the ground truth file it was compared against.
4.  **Store Results**: The final list of `TestResult` objects will be stored in the `Experiment` object.

### Part 3: Update Results Reporting

The final step is to update how the results are saved and displayed to account for the multiple scores.

  * **Update `TestResult.to_dict()`**: This method will be modified to include a new key, `ground_truth_source`, to identify which ground truth file the scores correspond to.
  * **Update CSV Output**: The main results summary CSV (`00_master_run_summary.csv`) will now potentially have multiple rows for a single experiment ID, with each row showing the scores against a different ground truth source. This provides a clear and detailed breakdown of the performance.

By following this plan, you can seamlessly integrate `LangExtract` and enhance the rigor of your project's evaluation framework.