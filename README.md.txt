databricks-medallion-etl-pipeline
Project Overview
This project demonstrates an end-to-end data engineering pipeline built on Databricks using Delta Lake and Apache Spark (PySpark). It implements a standard Medallion Architecture (Bronze, Silver, Gold layers) to process raw e-commerce sales data. The pipeline transforms raw, unvalidated data into a clean, structured, and aggregated format suitable for business intelligence and analytics.

The entire workflow is defined and deployed using Databricks Asset Bundles, showcasing best practices for reproducible, version-controlled deployments across different environments (e.g., development and production).

Architecture
The pipeline follows a Medallion Architecture pattern:

graph TD
    A[Raw Sales Data (CSV files in Cloud Storage)] -->|Auto Loader (Incremental Ingest)| B(Bronze Layer: `bronze_sales_table` - Raw Delta Table)
    B -->|Stream Processing (PySpark)| C(Silver Layer: `silver_sales_table` - Cleaned & Standardized Delta Table)
    C -->|Stream Processing (PySpark Aggregations)| D(Gold Layer: `gold_sales_summary_table` - Aggregated Delta Table)
    D --> E[Business Intelligence / Analytics Tools]

Key components and data flow:

Raw Sales Data: Simulated as CSV files landing in a cloud storage directory (/FileStore/ecommerce_sales_project_<env>/raw_sales_data). The 00_Setup_and_Data_Generation.py notebook programmatically creates these files.

Bronze Layer (bronze_sales_table):

Ingestion: Data is ingested incrementally using Databricks Auto Loader, which efficiently discovers new files.

Storage: Stored as a Delta Lake table.

Content: Contains raw, unvalidated data as it arrives, along with ingestion metadata.

Silver Layer (silver_sales_table):

Transformation: Reads streaming data from the Bronze layer.

Cleaning & Standardization: Applies data type conversions, trims whitespace, and filters out malformed or invalid records (e.g., null IDs, non-positive quantities/prices).

Derived Attributes: Calculates total_amount and adds processing metadata.

Storage: Stored as a Delta Lake table.

Gold Layer (gold_sales_summary_table):

Aggregation: Reads streaming data from the Silver layer.

Business-Ready Data: Performs aggregations (e.g., daily total sales, total quantity sold per product per store) for reporting.

Storage: Stored as a Delta Lake table, optimized for analytical queries.

Technologies Used
Databricks: Unified analytics platform.

Delta Lake: Open-source storage layer providing ACID transactions, schema enforcement & evolution, and time travel.

Apache Spark (PySpark): For distributed data processing.

Spark Structured Streaming: For building continuous data pipelines.

Databricks Auto Loader: For efficient and incremental data ingestion from cloud storage.

Databricks Asset Bundles (DABs): For defining, deploying, and managing Databricks jobs and other resources as code.

DBFS (Databricks File System): Used for simulating cloud storage paths and storing project data.

Python: Programming language for notebooks.

Skills Demonstrated
This project showcases expertise in:

Data Ingestion: Efficiently ingesting incremental data using Auto Loader.

Data Transformation: Performing cleaning, type casting, and derived calculations with PySpark.

Data Modeling: Implementing a multi-layered (Bronze, Silver, Gold) data architecture.

Delta Lake Fundamentals: Creating and managing Delta tables, understanding ACID properties.

Streaming Data Processing: Building end-to-end streaming ETL pipelines with fault tolerance (via checkpoints).

Data Quality: Incorporating basic data validation and filtering.

Databricks Asset Bundles: Defining and deploying complex multi-task Databricks Jobs as code.

Reproducibility & Environment Management: Setting up and cleaning the environment for consistent runs across different deployment targets.

Spark SQL: Implicitly used by PySpark DataFrame operations and available for direct querying.

Project Setup and How to Run
To run this project in your Databricks workspace using Databricks Asset Bundles:

Prerequisites:
Databricks Workspace: Access to a Databricks workspace.

Databricks CLI: Install the Databricks CLI on your local machine (installation guide).

Authentication: Configure authentication for your Databricks CLI with your workspace. OAuth (User-to-Machine) is recommended for development:

databricks auth login --host https://<your-databricks-workspace-url>

(Replace <your-databricks-workspace-url> with your actual workspace URL, e.g., https://dbc-abcdefgh-ijkl.cloud.databricks.com). Follow the browser prompts.

Python 3.8+: Ensure you have Python installed locally.

Steps:
Clone this GitHub Repository Locally:

git clone https://github.com/<your-username>/databricks-medallion-etl-pipeline.git
cd databricks-medallion-etl-pipeline

Configure databricks.yml:

Open the databricks.yml file in the root of your cloned repository.

Crucially, replace the placeholder https://<your-databricks-workspace-url> with the actual URL of your Databricks workspace.

Review and adjust cluster_node_type_id and cluster_num_workers variables under both dev and prod targets if you have specific cluster requirements.

(Optional) Update email_notifications with your desired email address.

Validate the Bundle:

From your terminal, ensure you are in the root directory of your project (where databricks.yml is located).

Run the validation command:

databricks bundle validate

This checks for syntax errors and structural correctness of your bundle.

Deploy the Bundle to your Development Environment:

Deploy the bundle to your dev target:

databricks bundle deploy -t dev

This command will:

Upload all notebooks from the notebooks/ directory to your Databricks workspace (e.g., under /Users/<your-email>/ecommerce-sales-etl-bundle/dev/files/notebooks/).

Create or update a Databricks Job in the Workflows section of your Databricks UI. This job will be named dev - Ecommerce Sales ETL Pipeline and will contain sequential tasks for each notebook.

Run the Databricks Job:
You can initiate a run of the deployed job in two ways:

From Databricks UI:

Navigate to the Workflows section in your Databricks workspace.

Find the job named dev - Ecommerce Sales ETL Pipeline.

Click on the job and then click the "Run now" button.

From Databricks CLI:

databricks bundle run ecommerce_sales_etl_job -t dev

This command triggers a run of the specified job on the dev target.

Monitor Job Progress:

In the Databricks UI, under Workflows, click on the dev - Ecommerce Sales ETL Pipeline job. You will see its runs in the "Job runs" tab.

Click on an active run to see the status of each task (Setup, Bronze, Silver, Gold, Query & Cleanup). You can click on individual tasks to view their notebook outputs.

Understanding the Job Execution Flow:
When the Databricks Job runs:

00_Setup_and_Data_Generation.py will execute first. It creates the necessary DBFS folders and writes the mock sales_day_1.csv and sales_day_2.csv into the specified raw data input path (e.g., /FileStore/ecommerce_sales_project_dev/raw_sales_data). It also cleans up any previous runs to ensure a fresh start.

The streaming jobs defined in 01_Bronze_Layer.py, 02_Silver_Layer.py, and 03_Gold_Layer.py will then start sequentially. Since the job runs the notebooks, these streams will typically process any available data in micro-batches and continue to run until the job finishes or they are explicitly stopped.

Finally, 04_Query_and_Cleanup.py will run. This notebook performs example queries on your gold_sales_summary_table to show the aggregated results. Crucially, it also includes code to stop all active streaming jobs and clean up all created Delta tables and checkpoint directories in DBFS, leaving your workspace tidy after the demonstration.

Mock Data
The mock sales data is generated programmatically by 00_Setup_and_Data_Generation.py. It simulates sales transactions with transaction_id, customer_id, product_id, quantity, price_per_unit, transaction_timestamp, store_id, and payment_method. The data/raw_sales/ folder in this repository contains copies of these files for direct review.

Challenges and Solutions
Handling Raw Data Ingestion: Utilized Auto Loader for reliable and incremental ingestion of new files, avoiding manual file tracking. cloudFiles.schemaLocation ensures robust schema inference and evolution.

Data Type Inconsistencies: Initially ingested all potentially problematic columns (like quantity, price, timestamp) as StringType in the Bronze layer to prevent ingestion failures from malformed data. Type casting and validation are performed in the Silver layer.

Ensuring Data Quality: Implemented basic filtering in the Silver layer to drop or flag records with critical null values or invalid numeric ranges, ensuring cleaner data for downstream analysis. For production, more advanced frameworks like Great Expectations could be integrated.

Maintaining Historical State in Streams: For Gold layer aggregations, outputMode("complete") was chosen for simplicity to always show the latest aggregates. For large-scale, continuously growing aggregates, stateful streaming aggregations with watermarks or periodic MERGE operations in batch mode would be more suitable.

Resource Management: Implemented explicit stream stopping and comprehensive directory cleanup within the 04_Query_and_Cleanup.py notebook to manage Databricks cluster resources and DBFS storage effectively. This ensures a clean slate for repeated runs.

Reproducible Deployments: Leveraged Databricks Asset Bundles to define the entire project infrastructure (notebooks, job tasks, cluster configs, paths) as code, enabling consistent deployments across different environments and simplifying CI/CD.

Future Enhancements
Databricks Delta Live Tables (DLT): Refactor the Bronze, Silver, and Gold notebooks into a declarative DLT pipeline for simplified development, built-in data quality expectations, and automatic dependency management.

Advanced Data Quality: Integrate a dedicated data quality framework (e.g., Great Expectations) with more robust logging for rejected records and alerting.

Data Enrichment: Incorporate additional dimension tables (e.g., product_master, customer_details) and join them to enrich the data in the Silver or Gold layers.

SCD Type 2 Implementation: For dimension tables, implement Slowly Changing Dimension (SCD) Type 2 logic using Delta Lake's MERGE operations to track historical attribute changes.

Performance Optimization: Apply advanced Delta Lake optimizations like Z-ordering or Liquid Clustering on relevant Delta tables for improved query performance.

Monitoring and Alerting: Enhance job monitoring with more detailed logging, custom metrics, and integration with external alerting systems (e.g., PagerDuty, Slack).

CI/CD Pipeline Integration: Set up a full CI/CD pipeline using GitHub Actions to automate testing, validation, and deployment of the Databricks Asset Bundle to different environments upon code changes.

Unit/Integration Testing: Add PySpark unit tests for individual transformation logic and integration tests for the overall pipeline flow.