# Automated Excel to BigQuery Loading Pipeline

This script automates the process of taking an Excel file with multiple sheets from Google Cloud Storage (GCS), converting each sheet into a CSV file, storing these CSVs in a designated GCS bucket, and finally loading each CSV into a separate table in Google BigQuery. All configurations are done directly within the Colab notebook.

## Overview

The primary goal of this solution is to streamline the ingestion of multi-sheet Excel data into BigQuery. It handles:
1.  Reading an Excel file directly from a GCS bucket.
2.  Converting each sheet of the Excel file into an individual CSV file.
3.  Sanitizing column names from the Excel sheets to ensure BigQuery compatibility.
4.  Storing these generated CSVs temporarily in a GCS location.
5.  Copying the processed CSVs to a final GCS location.
6.  Loading each CSV file from its final GCS location into a distinct BigQuery table, with schema auto-detection.

This script is designed to be run in a Google Cloud Colab Enterprise environment or a similar Vertex AI Workbench (JupyterLab) environment that has access to your Google Cloud project resources and the necessary libraries.

## Features

* **Multi-Sheet Excel Processing:** Handles Excel files with multiple tabs, creating one CSV per tab.
* **GCS Integration:** Reads the source Excel file from GCS and writes intermediate and final CSVs back to GCS.
* **BigQuery Loading:** Loads data into BigQuery using `LOAD DATA OVERWRITE` SQL statements, ensuring tables are refreshed with new data.
* **Schema Autodetection:** Leverages BigQuery's ability to automatically detect the schema from CSV files.
* **Column Name Sanitization:** Automatically cleans up column headers from the Excel file to be compatible with BigQuery naming conventions.
* **Self-Contained Configuration:** All key parameters (GCS bucket names, file paths, BigQuery project ID, dataset ID, etc.) are set directly in **Code Cell 1** of the notebook.
* **Programmatic Execution:** The entire pipeline is orchestrated via a Python script suitable for Colab Enterprise notebooks.

## Prerequisites

Before running this script, ensure you have the following:

1.  **Google Cloud Project:** A Google Cloud Platform project.
2.  **Google Cloud Storage (GCS) Buckets:**
    * A bucket to store your source Excel file.
    * A bucket to store the final processed CSV files. This can be the same as the input bucket but using different folders is recommended.
3.  **BigQuery Dataset:** A BigQuery dataset must already exist in your project where the new tables will be created.
4.  **Colab Enterprise or Vertex AI Workbench Environment:** A notebook environment with the necessary Python libraries installed (see `Code Cell 0` in the notebook for imports: `pandas`, `google-cloud-storage`, `google-cloud-bigquery`, `re`). These are typically pre-installed in Google Cloud managed notebook environments.
5.  **IAM Permissions:** The service account associated with your Colab Enterprise notebook (or your user credentials if running locally with `gcloud auth application-default login`) needs the following IAM roles/permissions:
    * **GCS:**
        * `roles/storage.objectViewer` on the input GCS bucket (to read the Excel file).
        * `roles/storage.objectCreator` and `roles/storage.objectAdmin` (or at least `objectCreator` and `objectDeleter` if overwriting) on the buckets/folders used for temporary and final CSVs.
    * **BigQuery:**
        * `roles/bigquery.dataEditor` on the target dataset (to create/overwrite tables and load data).
        * `roles/bigquery.jobUser` on the project (to run BigQuery jobs).
        * `roles/bigquery.user` (provides permissions to run queries, list datasets, etc.)

## Setup & Configuration

1.  **Download or Copy the Notebook:** Get the Colab notebook (`.ipynb` file) which contains all the script cells.
2.  **Upload to Colab Enterprise / Vertex AI Workbench:** Upload the notebook to your environment.
3.  **Configure Variables in Code Cell 1:**
    * Open the notebook.
    * Navigate to **Code Cell 1 ("Environment Setup & User Configuration")**.
    * **Carefully update the placeholder/example values** in the "ACTION REQUIRED" section with your specific GCS bucket names, Excel filename, BigQuery project ID, dataset ID, etc.

## Running the Script

1.  **Open the Colab Notebook.**
2.  **Modify Code Cell 1:** Ensure all configuration variables in Code Cell 1 are correctly set for your environment.
3.  **Run the cells sequentially (Cell 0 through Cell 4 as described below).**
    * **Verify the printed "GCS Path Configuration Loaded" and "BigQuery Configuration Loaded" sections** after running Cell 1 to ensure your settings are correct before proceeding.
    * Monitor the output of each cell for any errors or warnings. Successful execution will result in new tables in your specified BigQuery dataset.

## Script Structure (Colab Notebook Cells)

The Colab notebook is structured into the following cells. You should run them in order.

* **Cell 0: Imports:** Imports all required Python packages.
* **Cell 1: Environment Setup & User Configuration:**
    * Sets up environment variables (like auto-detected `project_id`, `user`, timestamp).
    * **Contains all user-configurable variables** for GCS paths, BigQuery details, etc. This is the primary cell you will edit.
* **Cell 2: `excel_to_csv_all_sheets_gcs_only` function:**
    * Reads the specified Excel file from GCS.
    * Iterates through each sheet.
    * Sanitizes column names for BigQuery compatibility.
    * Saves each sheet as a CSV file to a temporary GCS location.
* **Cell 3: `copy_gcs_files_to_another_bucket` function:**
    * Copies the generated CSV files from the temporary GCS location to the final GCS destination bucket/folder.
* **Cell 3.5: `load_csvs_to_bigquery_with_sql_loop` function:**
    * Takes the list of GCS URIs for the final CSV files.
    * For each CSV URI:
        * Derives a BigQuery table name.
        * Constructs a `LOAD DATA OVERWRITE ... FROM FILES ...` SQL query.
        * Executes the query using the BigQuery Python client to load data into the target table, auto-detecting the schema.
* **Cell 4: Execution - Run the Process:**
    * Constructs full GCS paths based on the configuration from Cell 1.
    * Calls the functions from Cell 2, Cell 3, and Cell 3.5 in sequence to execute the entire pipeline.

## Customization

* **Table Naming:** The BigQuery table names are derived from the original Excel filename, the sheet name, and the `bigquery_table_prefix` defined in Code Cell 1.
* **Skipping Rows:** The `bigquery_skip_leading_rows` in Code Cell 1 controls how many rows are skipped at the top of each CSV during the BigQuery load.
* **Load Behavior:** The script uses `LOAD DATA OVERWRITE`. To append, modify the SQL in `load_csvs_to_bigquery_with_sql_loop` to use `LOAD DATA INTO ...`.

## Troubleshooting Common Issues

* **`NameError: name 're' is not defined` (or similar):** Ensure Code Cell 0 (Imports) has been run.
* **Permission Denied / Forbidden errors:** Verify IAM permissions for the notebook's service account.
* **BigQuery "Field name ... is not supported":** This should be resolved by the column sanitization in Code Cell 2. Ensure Cell 2 was run to generate new CSVs with clean headers.
* **BigQuery Dataset Not Found:** Ensure `bigquery_dataset_id` in Code Cell 1 is correct and the dataset exists.
* **Placeholder Values in Output:** If the script's output shows "PLACEHOLDER" or incorrect values for your configuration, double-check that you've updated all necessary variables in Code Cell 1 and re-run that cell.

---

This script provides a robust starting point for automating your Excel to BigQuery workflows.
