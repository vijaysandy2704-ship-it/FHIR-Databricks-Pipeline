*FHIR Data Ingestion & Medallion Lakehouse Pipeline

This repository implements an end-to-end Medallion Lakehouse architecture using PySpark, Delta Lake, and Databricks to ingest, process, version, and analyze FHIR healthcare records (Patient, Encounter, Observation, Condition).

*Medallion Architecture Overview

*Bronze Layer (01_raw_to_bronze.ipynb): 
  - Ingests raw JSON payloads from the public HAPI FHIR REST API.
  - Handles API pagination (up to 3 pages per resource).
  - Sequentially ingests resources in mandated order: Patient -> Encounter -> Observation -> Condition.
  - Injects audit metadata columns (`extraction_timestamp` and `api_url_or_params`).

*Silver Layer (02_bronze_to_silver.ipynb): 
  - Extracts domain attributes from raw JSON payloads.
  - Cleans data and removes duplicates based on domain primary keys.
  - Implements Slowly Changing Dimension Type 2 (SCD Type 2) tracking using Delta Lake MERGE operations (is_current, effective_start_date, effective_end_date).

*Gold Layer (03_silver_to_gold.ipynb)*: 
Aggregates active records (is_current = true) into a business-ready 'Patient 360' analytical model (gold_patient_360).
Summarizes encounter totals, observation counts, and diagnosed conditions per patient.

 
 Orchestration & Automated Workflow

The pipeline is fully automated and orchestrated using 'Databricks Workflows (FHIR_Medallion_Pipeline)'. The DAG runs tasks sequentially to maintain referential integrity:

1. Raw_to_Bronze (01_raw_to_bronze)
2. Bronze_to_Silver (02_bronze_to_silver - depends on Task 1)
3. Silver_to_Gold(03_silver_to_gold - depends on Task 2)

