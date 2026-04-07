# Banking Transactions Fraud Detection (Rule-Based)

## Project Overview
This project implements an **end-to-end data engineering pipeline** to detect fraudulent banking transactions using **rule-based logic**. It leverages Azure cloud services and modern data engineering practices to ingest, transform, and analyze transaction data at scale.

The pipeline is designed using the **Medallion Architecture (Bronze, Silver, Gold)** to ensure data quality, scalability, and maintainability. Fraud detection is performed using business-driven rules such as identifying high-value transactions and detecting multiple transactions within a short time window.

---

## Architecture

**End-to-End Flow:**
  
```
Source (CSV / Azure SQL)
        │
        ▼
Azure Data Factory (Ingestion & Orchestration)
        │
        ▼
Azure Data Lake Storage (Bronze → Silver → Gold)
        │
        ▼
Azure Databricks (PySpark Transformations)
        │
        ▼
Delta Lake + Unity Catalog
        │
        ▼
Power BI Dashboard (Visualization)
```


---

## Medallion Architecture

### Bronze Layer (Raw Data)
- Stores raw ingested transaction data
- No transformations applied

### Silver Layer (Cleaned Data)
- Data is cleaned and standardized
- Transformations include:
  - Handling null values  
  - Data type casting  
  - Removing duplicates  
  - Renaming columns
  - Selecting required columns  

### Gold Layer (Business Layer)
- Applies fraud detection rules
- Includes fraud flags for each transaction
- Produces analytics-ready gold tables
    - fact_all_transactions
    - fact_fraud_transactions
    - dim_customer_details

---

## Fraud Detection Rules

Two key rule-based checks are implemented:

### 1. High-Value Transactions
Transactions exceeding a defined threshold of high value are flagged as fraud.

### 2. Multiple Transactions (Window Function)
Multiple transactions by same user within a short time window is flagged as fraud.

---

## Tech Stack

Tools & technologies used:
- Azure SQL
- Azure Data Factory
- Azure Databricks
- Unity Catalog
- Delta Lake
- Power BI


---

## Azure SQL

Azure SQL Database is used as the primary source system in this project. It stores structured bank transaction data before it is ingested into the data pipeline.

**Purpose:**
- Acts as the source of truth for transaction data
- Stores raw transactional records in a structured format
- Enables seamless integration with Azure Data Factory (ADF) for ingestion
- Supports scalable and secure data storage


---

## Azure Databricks 

Azure Databricks is used as the core data processing engine to transform and analyze transaction data using PySpark.

**Implementation:**

- All transformations from Bronze → Silver → Gold are implemented using PySpark with OOP principles (classes, reusable functions, modular design).
- Data is stored in respective layers as external tables in ADLS.
- Pipeline logs are maintained in a managed table for monitoring and debugging.


---

## Delta Tables

Delta Lake is used for storing data across Bronze, Silver, and Gold layers, ensuring reliable and optimized data processing.

Key Benefits:
- ACID Transactions
- Schema Enforcement
- Time Travel (data versioning)
- Efficient MERGE operations for incremental loads
- Faster query performance using optimized storage and indexing

Delta tables ensure reliable and optimized data processing.


---

## Unity Catalog

Used for centralized data governance and access control.

**Components:**
* Metastore → Central metadata repository
* External Locations → Secure mapping of ADLS paths
* Credentials → Secure storage access (Managed Identity)
* Catalog & Schemas → Logical separation (bronze, silver, gold)

### Unity Catalog Setup

1. Access Connector (Managed Identity)
- Created an Access Connector for Databricks
- Used as a Managed Identity to access ADLS securely (no secrets or access tokens)

2. Role Assignment
- Assigned role: Storage Blob Data Contributor
- Scope: ADLS Storage Account

3. Metastore Creation
- Created and attached Unity Catalog Metastore
- Defined root storage path in ADLS

``` sql
abfss://<container>@<storage-account>.dfs.core.windows.net/
```

4. Storage Credential
- Created a storage credential to define how Unity Catalog accesses storage.

```sql
CREATE STORAGE CREDENTIAL my_credential
WITH AZURE_MANAGED_IDENTITY
```
→ Uses the Access Connector’s managed identity


5. Create External Location
- External locations provide access to specific paths in ADLS.

```sql
CREATE EXTERNAL LOCATION my_external_location
URL 'abfss://<container>@<storage-account>.dfs.core.windows.net/'
WITH (STORAGE CREDENTIAL my_credential)
```

6. Create Catalog & Schema
```sql
CREATE CATALOG catalog_name;
USE CATALOG catalog_name;

CREATE SCHEMA schema_name;
```

### Security Benefits
- No hardcoded credentials
- Role-based access control (RBAC)
- Fine-grained permissions (catalog/schema/table level)
- Centralized governance across workspace

---

## Azure Data Factory Pipelines

Azure Data Factory (ADF) is used as the orchestration and data integration service in this project. It automates the movement of data from the source system to the data lake and triggers downstream processing workflows.

### Approach 1: Sequential Pipeline
- Copy Activity (Azure SQL → ADLS Bronze)
- Databricks Notebook Activity (Ingest Bronze)
- Databricks Notebook Activity (Bronze → Silver)
- Databricks Notebook Activity (Silver → Gold)

👉 Simple and easy to manage

### Approach 2: Modular Pipeline + Databricks Workflow
- Copy Activity (Azure SQL → ADLS Bronze)
- ADF triggers a Databricks job

- Databricks Job executes multiple notebooks(tasks):
  - Ingestion (Bronze)
  - Transformation (Bronze → Silver) 
  - Fraud Detection (Silver → Gold)

👉 More scalable and production-ready

### Linked Services & Datasets

**Linked Services**
- Define connections to external systems.
- Used to connect ADF with:
    - Azure SQL Database (source)
    - ADLS Gen2 (storage)
    - Azure Databricks (compute)
 
**Datasets**
Represent the data structure within a linked service.
- Define:
    - Table (for SQL source)
    - File format & path (for ADLS)

---

## Power BI

Power BI is used for data visualization and reporting, enabling stakeholders to monitor fraud trends and transaction insights.

### Fraud Analysis Dashboard

<img width="1458" height="816" alt="Fraud Analysis Dashboard" src="https://github.com/user-attachments/assets/951f45fa-c119-4361-b470-43cc7438d84d" />



---

## Project Structure

```
banking-fraud-detection-rule-based/
│
├── data/
│   └── sample/
│       └── credit_card_transactions.csv
│
├── notebooks/
│   ├── 00-Configurations.ipynb
│   ├── 01-Ingest (Bronze Layer).ipynb
│   ├── 02-Bronze-to-Silver.ipynb
│   └── 03-Silver-to-Gold.ipynb
│
├── pipelines/
│   ├── ADF-Pipeline.json
│   └── ADF-Databricks-Pipeline.json
│
├── screenshots/
│   ├── ADF-Pipeline.png
│   ├── ADF-Databricks-Pipeline.png
│   ├── ADLS-Container.png
│   ├── Project-Resource-Group.png
│   └── Databricks-Job-Tasks.png
│   └── Azure SQL - DB.png
│   └── Pipeline Logs Table.png
│
└── README.md
```
---

## Key Features

- End-to-end data pipeline using Azure services
- Rule-based fraud detection logic
- Medallion Architecture implementation
- Scalable data processing using PySpark
- Delta Lake for optimized storage
- Unity Catalog for governance
- Modular and reusable notebook design
- Logging & audit tracking


---

## Future Improvements

- Implement incremental data loading (watermarking/CDC) to process only new and updated records
- Use Delta Lake upserts (MERGE) for efficient insert/update handling instead of full loads
- Integrate ML-based fraud detection models for advanced anomaly detection
- Optimize performance using partitioning, Z-ordering, and caching in Databricks
- Enable real-time processing using streaming (Event Hub / Kafka) for instant fraud detection


---

## Conclusion

This project represents a complete, real-world data engineering solution for fraud detection, built using modern cloud technologies and industry best practices. It covers the entire data lifecycle—from ingestion to transformation and analytics—using a scalable Medallion Architecture on Azure. With robust pipeline orchestration, secure data governance via Unity Catalog, and efficient processing using Databricks and Delta Lake, the system is designed to handle large-scale transactional data reliably. Overall, it demonstrates a production-ready approach that is scalable, maintainable, and well-aligned with modern enterprise data platforms.


---

## Author 
Rachana Chavan
