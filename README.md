### Mnemosyne Project Repository

This repository contains the source code and documentation for Mnemosyne, a system that uses Large Language Models (LLMs) to automatically convert large, complex documents into queryable Knowledge Graphs (KGs) to check for completeness and consistency.

**Problem:** Legal codes, technical manuals, and regulatory frameworks often develop inconsistencies, redundancies, and informational gaps over time, especially when drafted collaboratively by multiple authors. Traditional manual review processes are labor-intensive, prone to human error, and struggle to keep pace with evolving documents. In domains like municipal law, these flaws can lead to costly legal disputes and significant revenue losses.

**Solution:** Mnemosyne addresses this challenge by implementing a novel multi-stage pipeline that systematically transforms unstructured text into a structured, analyzable format. The system ingests source documents, segments them into semantically coherent chunks to manage LLM context windows, and uses an LLM to perform Named Entity Recognition (NER) and Relation Extraction (RE).

The extracted entities and their relationships are used to build a robust property graph in a Neo4j database. A key feature of the system is its two-tiered memory architecture, which uses a transient Short-Term Memory (STM) for batch processing and a persistent Long-Term Memory (LTM) for the final, consolidated knowledge graph. This structured KG enables the use of declarative queries (e.g., Cypher) to audit the source document for logical and structural flaws.

**Key Features:**
* LLM-driven Information Extraction (NER & RE) from unstructured text.
* Automated Knowledge Graph construction and refinement.
* wo-tiered memory architecture for efficient and scalable processing.
* End-to-end traceability, linking every graph element back to its source sentence.
* Initial case study on Pennsylvania township laws, with a methodology designed to be generalizable.

This project was developed in partial satisfaction of the requirements for the degree of Doctor of Engineering at The George Washington University.
