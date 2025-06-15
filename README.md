
# Databricks Medallion ETL Pipeline


## Project Overview

This project showcases a robust, end-to-end data engineering pipeline built on **Databricks** using **Delta Lake** and **Apache Spark (PySpark)**. It implements the industry-standard **Medallion Architecture** (Bronze, Silver, Gold layers) to transform raw e-commerce sales data into a clean, structured, and aggregated format ready for business intelligence and analytics.

The entire workflow is defined and deployed using **Databricks Asset Bundles (DABs)**, demonstrating best practices for reproducible, version-controlled deployments across various environments (e.g., development and production).

-----

## Architecture

The pipeline adheres to the **Medallion Architecture** pattern, ensuring data quality and transformation in distinct stages:

```mermaid
graph TD
    A[Raw Sales Data (CSV files in Cloud Storage)] -->|Auto Loader (Incremental Ingest)| B(Bronze Layer: `bronze_sales_table` - Raw Delta Table)
    B -->|Stream Processing (PySpark)| C(Silver Layer: `silver_sales_table` - Cleaned & Standardized Delta Table)
    C -->|Stream Processing (PySpark Aggregations)| D(Gold Layer: `gold_sales_summary_table` - Aggregated Delta Table)
    D --> E[Business Intelligence / Analytics Tools]
```

**Key components and data flow:**

  * **Raw Sales Data:** Simulated CSV files land in a cloud storage directory (`/FileStore/ecommerce_sales_project_<env>/raw_sales_data`). The `00_Setup_and_Data_Generation.py` notebook programmatically creates these files.
  * **Bronze Layer (`bronze_sales_table`):**
      * **Ingestion:** Data is incrementally ingested using **Databricks Auto Loader**, efficiently discovering new files.
      * **Content:** Contains raw, unvalidated data as it arrives, along with ingestion metadata. Stored as a **Delta Lake table**.
  * **Silver Layer (`silver_sales_table`):**
      * **Transformation:** Reads streaming data from the Bronze layer.
      * **Cleaning & Standardization:** Applies data type conversions, trims whitespace, and filters out malformed or invalid records (e.g., null IDs, non-positive quantities/prices).
      * **Derived Attributes:** Calculates `total_amount` and adds processing metadata. Stored as a **Delta Lake table**.
  * **Gold Layer (`gold_sales_summary_table`):**
      * **Aggregation:** Reads streaming data from the Silver layer.
      * **Business-Ready Data:** Performs aggregations (e.g., daily total sales, total quantity sold per product per store) optimized for reporting and analytical queries. Stored as a **Delta Lake table**.
  * **Business Intelligence / Analytics Tools:** The Gold layer serves as the source for downstream consumption.

-----

## Technologies Used

  * **Databricks:** Unified analytics platform.
  * **Delta Lake:** Open-source storage layer providing ACID transactions, schema enforcement & evolution, and time travel.
  * **Apache Spark (PySpark):** For distributed data processing.
  * **Spark Structured Streaming:** For building continuous data pipelines.
  * **Databricks Auto Loader:** For efficient and incremental data ingestion from cloud storage.
  * **Databricks Asset Bundles (DABs):** For defining, deploying, and managing Databricks jobs and other resources as code.
  * **DBFS (Databricks File System):** Used for simulating cloud storage paths and storing project data.
  * **Python:** Programming language for notebooks.

-----

## Skills Demonstrated

This project effectively showcases expertise in:

  * **Data Ingestion:** Efficiently ingesting incremental data using **Auto Loader**.
  * **Data Transformation:** Performing cleaning, type casting, and derived calculations with **PySpark**.
  * **Data Modeling:** Implementing a multi-layered (Bronze, Silver, Gold) data architecture.
  * **Delta Lake Fundamentals:** Creating and managing Delta tables, understanding ACID properties.
  * **Streaming Data Processing:** Building end-to-end streaming ETL pipelines with fault tolerance (via checkpoints).
  * **Data Quality:** Incorporating basic data validation and filtering.
  * **Databricks Asset Bundles:** Defining and deploying complex multi-task Databricks Jobs as code.
  * **Reproducibility & Environment Management:** Setting up and cleaning the environment for consistent runs.

-----

## Project Setup and How to Run

To deploy and run this project in your Databricks workspace using Databricks Asset Bundles:

### Prerequisites

  * **Databricks Workspace:** Access to a Databricks workspace.
  * **Databricks CLI:** Install the Databricks CLI on your local machine ([installation guide](https://docs.databricks.com/en/dev-tools/cli/index.html)).
  * **Authentication:** Configure authentication for your Databricks CLI with your workspace. **OAuth (User-to-Machine) is recommended for development:**
    ```bash
    databricks auth login --host https://<your-databricks-workspace-url>
    ```
    (Replace `<your-databricks-workspace-url>` with your actual workspace URL, e.g., `https://dbc-abcdefgh-ijkl.cloud.databricks.com`). Follow the browser prompts.
  * **Python 3.8+:** Ensure you have Python installed locally.

### Steps

1.  **Clone this GitHub Repository Locally:**
    ```bash
    git clone https://github.com/<your-username>/databricks-medallion-etl-pipeline.git
    cd databricks-medallion-etl-pipeline
    ```
2.  **Configure `databricks.yml`:**
      * Open the `databricks.yml` file in the root of your cloned repository.
      * **Crucially, replace the placeholder `https://<your-databricks-workspace-url>` with the actual URL of your Databricks workspace.**
      * Review and adjust `cluster_node_type_id` and `cluster_num_workers` variables under both `dev` and `prod` targets if you have specific cluster requirements.
      * (Optional) Update `email_notifications` with your desired email address.
3.  **Validate the Bundle:**
    From your terminal (in the project root directory):
    ```bash
    databricks bundle validate
    ```
    This checks for syntax errors and structural correctness of your bundle.
4.  **Deploy the Bundle to your Development Environment:**
    ```bash
    databricks bundle deploy -t dev
    ```
    This command will:
      * Upload all notebooks from the `notebooks/` directory to your Databricks workspace.
      * Create or update a Databricks Job named `dev - Ecommerce Sales ETL Pipeline` in the Workflows section of your Databricks UI.
5.  **Run the Databricks Job:**
    You can initiate a run of the deployed job in two ways:
      * **From Databricks UI:** Navigate to the **Workflows** section, find the job, and click "Run now."
      * **From Databricks CLI:**
        ```bash
        databricks bundle run ecommerce_sales_etl_job -t dev
        ```
6.  **Monitor Job Progress:**
    In the Databricks UI, under **Workflows**, click on the `dev - Ecommerce Sales ETL Pipeline` job to view its runs and status. Click on individual tasks to see notebook outputs.

### Understanding the Job Execution Flow

When the Databricks Job runs:

1.  **`00_Setup_and_Data_Generation.py`** executes first. It creates necessary DBFS folders, writes mock `sales_day_1.csv` and `sales_day_2.csv` to the raw data input path, and cleans up any previous runs.
2.  The streaming jobs defined in **`01_Bronze_Layer.py`**, **`02_Silver_Layer.py`**, and **`03_Gold_Layer.py`** start sequentially. They process available data incrementally in micro-batches.
3.  Finally, **`04_Query_and_Cleanup.py`** runs. This notebook performs example queries on your `gold_sales_summary_table` and **crucially includes code to stop all active streaming jobs and clean up all created Delta tables and checkpoint directories**, ensuring a tidy workspace after the demonstration.

-----

## Mock Data

The mock sales data is programmatically generated by `00_Setup_and_Data_Generation.py`. It simulates sales transactions with `transaction_id`, `customer_id`, `product_id`, `quantity`, `price_per_unit`, `transaction_timestamp`, `store_id`, and `payment_method`. Copies of these files are available in the `data/raw_sales/` folder for review.

-----

## Challenges and Solutions

This project addresses common data engineering challenges:

  * **Handling Raw Data Ingestion:** Utilized **Auto Loader** for reliable, incremental ingestion and `cloudFiles.schemaLocation` for robust schema inference and evolution.
  * **Data Type Inconsistencies:** Initially ingested potentially problematic columns as `StringType` in the Bronze layer, performing type casting and validation in the Silver layer to prevent ingestion failures.
  * **Ensuring Data Quality:** Implemented basic filtering in the Silver layer to drop or flag records with critical null values or invalid numeric ranges.
  * **Maintaining Historical State in Streams:** For Gold layer aggregations, `outputMode("complete")` was chosen for simplicity. For large-scale production, stateful aggregations with watermarks or periodic `MERGE` operations would be more suitable.
  * **Resource Management:** Explicit stream stopping and comprehensive directory cleanup are implemented in the final notebook to manage Databricks cluster resources and DBFS storage effectively.
  * **Reproducible Deployments:** Leveraged **Databricks Asset Bundles** to define the entire project infrastructure as code, enabling consistent deployments across environments and simplifying CI/CD.

-----

## Future Enhancements

  * **Databricks Delta Live Tables (DLT):** Refactor into a declarative DLT pipeline for simplified development, built-in data quality expectations, and automatic dependency management.
  * **Advanced Data Quality:** Integrate a dedicated data quality framework (e.g., Great Expectations) with robust logging for rejected records and alerting.
  * **Data Enrichment:** Incorporate additional dimension tables (e.g., `product_master`, `customer_details`) and join them to enrich data in the Silver or Gold layers.
  * **SCD Type 2 Implementation:** For dimension tables, implement Slowly Changing Dimension (SCD) Type 2 logic using Delta Lake's `MERGE` operations.
  * **Performance Optimization:** Apply advanced Delta Lake optimizations like Z-ordering or Liquid Clustering on relevant tables.
  * **Monitoring and Alerting:** Enhance job monitoring with detailed logging, custom metrics, and integration with external alerting systems.
  * **CI/CD Pipeline Integration:** Set up a full CI/CD pipeline using GitHub Actions to automate testing, validation, and deployment of the Databricks Asset Bundle.
  * **Unit/Integration Testing:** Add PySpark unit tests for individual transformation logic and integration tests for the overall pipeline flow.

-----

✍️ Author
Suvajit Bhattacharjee - https://www.linkedin.com/in/suvajitbhattacharjee/
