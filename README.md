# MediNexus — Metadata-Driven Azure Data Engineering Platform

> An end-to-end, metadata-driven healthcare data engineering platform built using Azure Data Factory, Azure Data Lake Storage Gen2, Azure Databricks, Delta Lake, Unity Catalog, Azure SQL Database, Azure Key Vault, Logic Apps, and Databricks analytical dashboards.

---

## Project Overview

**MediNexus** is an end-to-end, metadata-driven Azure Data Engineering platform designed to demonstrate how healthcare data from multiple source systems can be ingested, transformed, governed, monitored, and delivered as analytics-ready datasets.

The platform implements a **Medallion Architecture** with clearly separated processing layers:

- **Landing** — raw source data is ingested into a common landing zone.
- **Bronze** — source data is persisted in Delta format with minimal transformation.
- **Silver** — data is standardized, validated, and transformed into clean, current-state datasets.
- **Gold** — curated datasets are created for healthcare analytics and reporting.

**Azure Data Factory (ADF)** provides orchestration and metadata-driven ingestion, while **Azure Databricks, Apache Spark, and Delta Lake** provide scalable data transformation and processing.

A central **Azure SQL Database control framework** drives the processing logic instead of requiring a separate hard-coded pipeline for each table. The metadata configuration determines:

- Which source tables and files are active
- Which source system should be processed
- Source and target table information
- Full or incremental load strategy
- Watermark columns for incremental processing
- Merge keys for applicable datasets
- Bronze, Silver, and Gold target definitions
- Processing stage and execution configuration

The platform also maintains dedicated **watermark and audit information** to support incremental processing and operational monitoring.

This metadata-driven design allows the same reusable pipeline framework to process multiple healthcare datasets and makes onboarding additional datasets primarily a configuration task rather than a new pipeline-development task.

## Architecture
MediNexus follows a metadata-driven Azure data engineering architecture that separates source ingestion, data transformation, governance, orchestration, and analytics consumption.

![MediNexus Architecture](./screenshots/architecture/medinexus-architecture.png)

he platform follows the flow:

Data Sources → Azure Data Factory → Landing → Bronze → Silver → Gold → Analytics

with supporting services for metadata management, governance, security, monitoring, and source control.

Architecture Components

Data Sources

Neon PostgreSQL — provides the internal healthcare data for hospitals, doctors, and patients.

ADLS Gen2 / CSV partner feeds — provides external lab_results and insurance_providers data.

Data Ingestion

Azure Data Factory provides the metadata-driven ingestion framework.

The same reusable pipeline processes multiple source tables and files based on configuration stored in Azure SQL Database.

Metadata & Orchestration

Azure SQL Database stores the control metadata, watermark information, and audit records.

Azure Data Factory orchestrates the complete workflow and passes a common adf_run_id across pipeline stages for traceable execution.

Data Transformation

Data is processed using Azure Databricks, Apache Spark, PySpark, and Delta Lake.

The Medallion Architecture separates processing into:

Bronze — source data persisted with minimal transformation.

Silver — standardized and validated data.

Gold — curated datasets designed for analytics and reporting.

Governance & Security

Unity Catalog provides centralized data governance and access control.

Azure Key Vault supports secure management of secrets and credentials.

Azure Logic Apps is used for automated email notification after pipeline execution.

Source Control / CI-CD

GitHub is used to version-control the Azure Data Factory and Databricks project assets.

Consumption

Curated Gold datasets are consumed through Databricks analytical dashboards and Databricks Genie interfaces.

End-to-End Data Flow

Source data is available in PostgreSQL and external CSV feeds.

Azure Data Factory reads the active metadata configuration from Azure SQL Database.

The appropriate source data is copied into the common Landing layer.

Databricks processes the landed data into the Bronze layer.

Bronze data is transformed into standardized Silver Delta tables.

Silver data is used to create curated Gold data products.

Gold datasets are consumed through Databricks analytics and Genie.

Pipeline execution is recorded in the audit framework, and the execution status is communicated through Logic Apps.
---

# Business Problem

Healthcare organizations often receive data from multiple systems and external partners.

For example:

- Hospital information may exist in a PostgreSQL database.
- Patient and doctor information may come from an operational database.
- Laboratory results may arrive as CSV files.
- Insurance provider information may arrive from another external source.

Without a centralized data platform, organizations can face:

- Manual data movement
- Duplicate pipelines
- Inconsistent transformations
- Difficult monitoring
- Limited data lineage
- Delayed reporting
- Difficult onboarding of new sources
- Poor visibility into pipeline failures

MediNexus addresses these challenges by implementing a centralized, metadata-driven Azure Data Engineering architecture.

---

# Key Objectives

The main objectives of MediNexus are:

1. Build an end-to-end Azure Data Engineering pipeline.
2. Implement metadata-driven ingestion.
3. Support multiple source systems.
4. Support Full, Append, and Merge processing.
5. Implement incremental processing using watermarks.
6. Build a Medallion Architecture.
7. Store processed data using Delta Lake.
8. Implement governance using Unity Catalog.
9. Implement centralized audit logging.
10. Build analytics-ready Gold datasets.
11. Implement pipeline success/failure notification.
12. Provide healthcare analytics through dashboards.
13. Establish Git-based source control.
14. Build a foundation for CI/CD implementation.

---

# Technology Stack

| Technology | Purpose |
|---|---|
| Azure Data Factory | Pipeline orchestration and metadata-driven ingestion |
| Azure Data Lake Storage Gen2 | Landing and data storage |
| Azure Databricks | Data transformation and processing |
| Apache Spark / PySpark | Distributed data processing |
| Delta Lake | Reliable transactional data storage |
| Unity Catalog | Data governance and access control |
| Azure SQL Database | Metadata, watermark and audit management |
| Azure Key Vault | Secure secret management |
| Azure Logic Apps | Email notification workflow |
| GitHub | Source control and repository management |
| Databricks Dashboards | Healthcare analytics |
| Databricks Genie | Natural-language analytics |

---

# Source Systems

MediNexus currently integrates data from multiple sources.

## 1. Neon PostgreSQL

Operational healthcare data is sourced from PostgreSQL tables such as:

- `hospitals`
- `doctors`
- `patients`

Example:

```text
Neon PostgreSQL
       |
       v
Azure Data Factory
       |
       v
ADLS Landing
       |
       v
Databricks Bronze
       |
       v
Databricks Silver
```

---

## 2. ADLS CSV Sources

External healthcare data is represented through CSV datasets such as:

- `lab_results`
- `insurance_providers`

These files are processed through the same metadata-driven ingestion framework.

```text
CSV Files
   |
   v
ADLS Landing
   |
   v
Bronze
   |
   v
Silver
```

---

# Medallion Architecture

MediNexus follows a Medallion Architecture consisting of:

```text
Source
  |
  v
Landing
  |
  v
Bronze
  |
  v
Silver
  |
  v
Gold
```

---

## Landing Layer

The Landing layer stores incoming source data before transformation.

The platform organizes landing data by source system, source table, and ingestion execution context.

Example concept:

```text
landing/
├── neon_postgres/
│   ├── hospitals/
│   ├── doctors/
│   └── patients/
│
└── adls_csv/
    ├── lab_results/
    └── insurance_providers/
```

The Landing layer provides a raw ingestion boundary between the source systems and Databricks processing.

---

# Bronze Layer

The Bronze layer stores the ingested data in Delta format.

The objective of Bronze is to preserve the incoming data while providing a reliable Delta-based processing layer.

Current Bronze datasets include:

- `hospitals`
- `doctors`
- `patients`
- `lab_results`
- `insurance_providers`

Bronze processing is intentionally separated from business-level transformations.

---

# Silver Layer

The Silver layer contains cleaned and transformed data suitable for analytical processing.

Silver processing includes:

- Data transformation
- Incremental processing
- Deduplication
- Merge/upsert processing
- Append processing
- Full refresh processing
- Data quality validation
- Silver audit timestamps

The Silver layer contains the primary business-ready datasets used by the Gold transformations.

---

# Gold Layer

The Gold layer contains analytics-ready datasets designed for business consumption.

MediNexus currently produces three major Gold datasets:

### 1. Patient 360

`patient_360`

Provides a consolidated patient-level view containing information such as:

- Patient information
- Hospital association
- Insurance information
- Total laboratory tests
- Abnormal tests
- Critical tests
- Latest test date
- Patient age

The dataset maintains one analytical record per patient.

---

### 2. Hospital Daily Summary

`hospital_daily_summary`

Provides hospital-level analytical metrics such as:

- Hospital
- Test date
- Day name
- Weekend indicator
- Total tests
- Unique patients
- Active doctors
- Abnormal tests
- Critical tests
- Previous available day metrics
- Test volume changes

Window functions such as `LAG()` are used for previous-period comparisons.

---

### 3. Lab Test Trends

`lab_test_trends`

Provides laboratory testing trends across time.

Metrics include:

- Test month
- Test name
- Total tests
- Abnormal tests
- Critical tests
- Abnormal rate
- Previous-month abnormal rate
- Abnormal rate change

The month-over-month abnormal rate change is represented as a **percentage-point difference**.

---

# Metadata-Driven Framework

One of the core features of MediNexus is its metadata-driven architecture.

Instead of hard-coding every table into the orchestration framework, configuration is stored in Azure SQL Database.

The primary metadata table is:

```text
ctrl.table_config
```

The configuration determines how each dataset should be processed.

Example configuration concepts:

```text
table_id
source_system
source_schema_name
source_table_name
source_file_format
source_path
bronze_table_name
silver_table_name
gold_table_name
load_type
watermark_column
merge_keys
stage
is_active
```

---

# Load Types

The framework currently supports multiple loading strategies.

## FULL

Used when the complete dataset is loaded.

Example:

```text
Source
  |
  v
Landing
  |
  v
Bronze
  |
  v
Silver overwrite
```

---

## APPEND

Used for datasets where new records are added over time.

Example:

```text
Existing Silver
      +
New Records
      |
      v
Updated Silver
```

`lab_results` currently uses an append-oriented incremental pattern.

---

## MERGE

Used when existing records may be updated and new records may be inserted.

Example:

```text
Source
  |
  v
Watermark Filter
  |
  v
Silver MERGE
  |
  +---- Existing key -> UPDATE
  |
  +---- New key ------> INSERT
```

The `patients` dataset currently uses a MERGE strategy.

---

# Incremental Processing

MediNexus implements incremental processing using watermarks.

The watermark metadata is maintained in:

```text
ctrl.watermark
```

Example concepts:

```text
table_id
watermark_value
last_updated_timestamp
```

For incremental datasets, the pipeline uses a configured source column to identify new or changed records.

For example:

```text
patients
    |
    +-- watermark_column = updated_at
    |
    +-- merge_keys = patient_id
```

This allows the pipeline to avoid unnecessarily processing the entire source dataset on every execution.

---

# Metadata Control Tables

The framework uses centralized Azure SQL control tables.

## 1. `ctrl.table_config`

Controls how each dataset should be processed.

Responsibilities:

- Source configuration
- Target configuration
- Load type
- Watermark configuration
- Merge keys
- Pipeline stage
- Active/inactive status

---

## 2. `ctrl.watermark`

Maintains incremental processing state.

Responsibilities:

- Current watermark
- Last watermark update information

---

## 3. `ctrl.audit_log`

Tracks pipeline execution.

Example fields include:

```text
audit_id
adf_run_id
table_id
stage
status
records_written
start_time
end_time
created_timestamp
```

This provides table-level execution visibility.

---

# Azure Data Factory Architecture

ADF acts as the orchestration layer.

The main orchestration pipeline is:

```text
PL_MASTER
```

It coordinates:

```text
                    PL_MASTER
                        |
        +---------------+---------------+
        |                               |
        v                               v
SOURCE_TO_SILVER                 SOURCE_TO_SILVER
   PostgreSQL                       ADLS CSV
        |                               |
        +---------------+---------------+
                        |
                        v
                 SILVER TO GOLD
```

The PostgreSQL and CSV processing branches can run independently before the Gold processing stage.

Gold processing starts after the required upstream branches successfully complete.

---

# ADF Pipeline Hierarchy

## PL_MASTER

Master orchestration pipeline.

Responsibilities:

- Start source ingestion
- Execute PostgreSQL ingestion
- Execute CSV ingestion
- Wait for upstream processing
- Start Gold processing

---

## PL_SOURCE_TO_SILVER

Metadata-driven outer pipeline.

Responsibilities:

1. Query active metadata.
2. Identify tables for the selected source system.
3. Iterate through the metadata.
4. Invoke the inner processing pipeline.

---

## PL_SOURCE_TO_SILVER_INNER

Per-table processing pipeline.

Responsibilities include:

- Read table metadata
- Set execution start time
- Determine load type
- Read watermark when required
- Select source-specific processing
- Copy data to Landing
- Execute Landing → Bronze
- Execute Bronze → Silver
- Write audit information
- Update watermark after successful processing

---

## Silver to GOLD

Gold orchestration pipeline.

It reads the Gold-stage metadata and dynamically invokes the appropriate Databricks notebook.

Example:

```text
table_config
     |
     v
gold_table_name
     |
     v
Dynamic Notebook Path
     |
     +--> NB_patient_360
     |
     +--> NB_hospital_daily_summary
     |
     +--> NB_lab_test_trends
```

---

# Databricks Architecture

Databricks is responsible for distributed data processing and transformation.

The workspace is organized using Unity Catalog.

Conceptually:

```text
Catalog
└── adb_medinexus
    ├── landing
    ├── bronze
    ├── silver
    └── gold
```

The project uses Delta tables throughout the processing layers.

---

# Unity Catalog

Unity Catalog provides the governance layer for the Databricks environment.

MediNexus uses Unity Catalog for:

- Catalog organization
- Schema organization
- Data access governance
- Managed metadata
- External storage integration

The project also uses an external volume for the Landing layer.

Example:

```text
adb_medinexus.landing.landing_volume
```

---

# Delta Lake

Delta Lake is used as the storage format for the Bronze, Silver, and Gold datasets.

Benefits demonstrated in the project include:

- ACID transactions
- Reliable table updates
- Schema enforcement
- Schema evolution capabilities
- Time travel
- Transaction history
- MERGE operations
- Transaction logs

Delta transaction history is maintained through the `_delta_log` directory.

---

# Data Quality Validation

Data quality checks were performed across the Medallion layers.

Examples include:

### Key validation

- Null primary/business keys
- Duplicate business keys

### Record-count validation

- Source-to-Bronze counts
- Bronze-to-Silver counts
- Silver-to-Gold reconciliation

### Metric reconciliation

Gold metrics were independently reconciled against Silver data.

Examples:

- Total laboratory tests
- Abnormal tests
- Critical tests
- Unique patients
- Active doctors
- Monthly trends

---

# Data Validation Results

The current project validation includes:

| Dataset | Records |
|---|---:|
| Bronze hospitals | 50 |
| Bronze doctors | 500 |
| Bronze patients | 5,000 |
| Bronze lab_results | 20 |
| Bronze insurance_providers | 30 |
| Silver hospitals | 50 |
| Silver doctors | 500 |
| Silver patients | 5,000 |
| Silver lab_results | 20 |
| Silver insurance_providers | 30 |
| Gold patient_360 | 5,000 |
| Gold hospital_daily_summary | 20 |
| Gold lab_test_trends | 8 |

The Gold datasets operate at different grains, so their row counts should not be expected to match the Silver source row counts directly.

---

# Healthcare Analytics Dashboard

MediNexus includes a Databricks Healthcare Analytics Dashboard.

The dashboard is organized into analytical views such as:

1. Executive Healthcare Overview
2. Patient Analytics
3. Hospital Performance
4. Laboratory Analytics

<!-- SCREENSHOT PLACEHOLDER
Add screenshot of Executive Healthcare Overview here.

Suggested file:
docs/images/dashboard-executive-overview.png
-->

```text
[ SCREENSHOT — EXECUTIVE HEALTHCARE OVERVIEW ]
```

<!-- SCREENSHOT PLACEHOLDER
Add screenshot of Patient Analytics here.

Suggested file:
docs/images/dashboard-patient-analytics.png
-->

```text
[ SCREENSHOT — PATIENT ANALYTICS ]
```

<!-- SCREENSHOT PLACEHOLDER
Add screenshot of Hospital Performance here.

Suggested file:
docs/images/dashboard-hospital-performance.png
-->

```text
[ SCREENSHOT — HOSPITAL PERFORMANCE ]
```

<!-- SCREENSHOT PLACEHOLDER
Add screenshot of Laboratory Analytics here.

Suggested file:
docs/images/dashboard-laboratory-analytics.png
-->

```text
[ SCREENSHOT — LABORATORY ANALYTICS ]
```

---

# Databricks Genie

MediNexus also includes a Databricks Genie experience for natural-language analytics.

The objective is to allow users to ask questions about the healthcare datasets without manually writing SQL.

Example:

```text
Question:
"I want state wise patient count"

Result:
Patient distribution by state
```

<!-- SCREENSHOT PLACEHOLDER
Add Genie screenshot here.

Suggested file:
docs/images/databricks-genie.png
-->

```text
[ SCREENSHOT — DATABRICKS GENIE ]
```

---

# Pipeline Monitoring and Audit

Pipeline execution is tracked using the centralized audit framework.

Each processed table can generate an audit record containing:

```text
ADF Run ID
Table ID
Stage
Status
Records Written
Start Time
End Time
```

This provides visibility into individual table processing rather than relying only on the overall pipeline status.

---

# Email Notification

MediNexus uses an Azure Logic App for pipeline notification.

The architecture is:

```text
Azure Data Factory
       |
       | HTTP POST
       v
Azure Logic App
       |
       v
Email Notification
```

The notification can contain:

- Pipeline status
- Run information
- Table-level processing status
- Records written
- Execution information

<!-- SCREENSHOT PLACEHOLDER
Add successful MediNexus email notification screenshot.

Suggested file:
docs/images/pipeline-success-email.png
-->

```text
[ SCREENSHOT — SUCCESS EMAIL ]
```

---

# Security and Governance

Security-related Azure services are incorporated into the platform.

## Azure Key Vault

Key Vault is used as the secure location for secrets and credentials required by the data platform.

The architecture avoids embedding sensitive credentials directly into pipeline logic where possible.

---

## Unity Catalog

Unity Catalog provides centralized governance for Databricks data assets.

---

## Azure SQL Control Database

Metadata and audit information are maintained separately from the analytical data.

This provides a centralized control plane for the ingestion framework.

---

# GitHub Source Control

MediNexus uses separate repositories for different responsibilities.

### Azure Data Factory

Repository:

```text
medinexus-azure-data-factory
```

Contains ADF-related source-controlled artifacts such as:

```text
dataset/
factory/
linkedService/
pipeline/
trigger/
README.md
publish_config.json
```

---

### Azure Databricks

Repository:

```text
medinexus-adb-repo
```

Contains Databricks notebooks and related Databricks project assets.

---

### Portfolio Repository

Repository:

```text
medinexus-azure-data-platform
```

This repository is intended for:

- Architecture documentation
- Project explanation
- Screenshots
- Design decisions
- Implementation notes
- Data validation evidence
- CI/CD documentation
- Interview reference material

The portfolio repository is intentionally separated from the application/source repositories.

---

# Repository Architecture

```text
GitHub
│
├── medinexus-azure-data-factory
│   └── Azure Data Factory source control
│
├── medinexus-adb-repo
│   └── Azure Databricks source control
│
└── medinexus-azure-data-platform
    ├── README.md
    ├── docs/
    ├── architecture/
    ├── screenshots/
    └── documentation/
```

---

# CI/CD

MediNexus is being developed with Git-based source control and a separation between:

- ADF source code
- Databricks source code
- Portfolio documentation

The next phase of the project is to implement and document a more complete CI/CD workflow for the platform.

The intended direction is:

```text
Developer
    |
    v
Feature Branch
    |
    v
Pull Request
    |
    v
Code Review
    |
    v
Main Branch
    |
    v
Deployment Pipeline
    |
    +---------> Azure Data Factory
    |
    +---------> Azure Databricks
```

> **Status:** Git-based source control is established. Full automated CI/CD is a work in progress and will be documented here as it is implemented.

---

# Current Data Processing Configuration

The current metadata framework contains configurations for:

| Source | Dataset | Load Type | Incremental Column | Merge Key |
|---|---|---|---|---|
| Neon PostgreSQL | hospitals | FULL | — | — |
| Neon PostgreSQL | doctors | FULL | — | — |
| Neon PostgreSQL | patients | MERGE | `updated_at` | `patient_id` |
| ADLS CSV | lab_results | APPEND | `reported_at` | — |
| ADLS CSV | insurance_providers | FULL | — | — |

Gold processing includes:

| Gold Dataset | Processing |
|---|---|
| patient_360 | FULL |
| hospital_daily_summary | FULL |
| lab_test_trends | FULL |

---

# Project Data Flow

The complete processing flow can be summarized as:

```text
                     SOURCE
                       |
          +------------+------------+
          |                         |
   PostgreSQL                    CSV Files
          |                         |
          +------------+------------+
                       |
                       v
                Azure Data Factory
                       |
                       v
                 Metadata Lookup
                       |
                       v
                    Landing
                       |
                       v
                    Bronze
                       |
                       v
                    Silver
                       |
              +--------+--------+
              |        |       |
              v        v       v
          Patient   Hospital   Lab
            360      Summary   Trends
              \        |       /
               \       |      /
                +------+-----+
                       |
                       v
                Analytics Layer
                       |
          +------------+------------+
          |                         |
      Dashboard                  Genie
```

---

# Error Handling and Monitoring

The platform is designed to provide operational visibility through:

- ADF pipeline execution status
- Table-level audit logging
- Metadata-driven execution
- Watermark tracking
- Logic App notifications
- Databricks notebook execution monitoring

This allows failures to be investigated at the table/stage level rather than treating the entire platform as a single black box.

---

# Performance and Spark Optimization

The Databricks implementation also demonstrates Spark optimization concepts.

Techniques explored in the project include:

- Broadcast joins
- Data skew mitigation
- Z-ORDER
- Efficient Delta processing
- Incremental processing
- Partition-aware processing
- Idempotent pipeline design

These techniques are particularly relevant when processing larger datasets in production-scale Spark environments.

---

# Idempotency

The pipeline design aims to support repeatable processing.

The use of:

- Metadata configuration
- Controlled landing paths
- Delta tables
- MERGE operations
- Audit logging
- Watermark-based incremental processing

helps make pipeline execution more predictable and easier to recover.

---

# Current Validation and Testing

The platform has been tested through multiple validation layers.

### Pipeline validation

- ADF master pipeline execution
- Source-to-Silver execution
- Silver-to-Gold execution
- Audit log verification
- Email notification verification

### Data validation

- Record counts
- Null checks
- Duplicate checks
- Business-key validation
- Gold metric reconciliation
- Monthly trend validation
- Hospital-day metric validation

### Analytical validation

- Patient 360 metrics
- Hospital summary metrics
- Laboratory trend calculations
- Window-function calculations
- Percentage-point changes

---

# Screenshots

This section will be expanded as the project documentation is finalized.

## Architecture

<!-- ADD SCREENSHOT HERE -->

```text
[ Architecture Diagram ]
```

---

## Azure Resource Group

<!-- ADD SCREENSHOT HERE -->

```text
[ Azure Resource Group Screenshot ]
```

---

## Azure Data Factory

### PL_MASTER

<!-- ADD SCREENSHOT HERE -->

```text
[ PL_MASTER Screenshot ]
```

### Source-to-Silver Pipeline

<!-- ADD SCREENSHOT HERE -->

```text
[ SOURCE_TO_SILVER Screenshot ]
```

### Silver-to-Gold Pipeline

<!-- ADD SCREENSHOT HERE -->

```text
[ SILVER_TO_GOLD Screenshot ]
```

---

## Metadata Control Tables

### table_config

<!-- ADD SCREENSHOT HERE -->

```text
[ ctrl.table_config Screenshot ]
```

### watermark

<!-- ADD SCREENSHOT HERE -->

```text
[ ctrl.watermark Screenshot ]
```

### audit_log

<!-- ADD SCREENSHOT HERE -->

```text
[ ctrl.audit_log Screenshot ]
```

---

## Databricks

### Unity Catalog

<!-- ADD SCREENSHOT HERE -->

```text
[ Unity Catalog Screenshot ]
```

### Bronze

<!-- ADD SCREENSHOT HERE -->

```text
[ Bronze Screenshot ]
```

### Silver

<!-- ADD SCREENSHOT HERE -->

```text
[ Silver Screenshot ]
```

### Gold

<!-- ADD SCREENSHOT HERE -->

```text
[ Gold Screenshot ]
```

---

## Dashboard

<!-- ADD SCREENSHOT HERE -->

```text
[ Dashboard Screenshot ]
```

---

## Databricks Genie

<!-- ADD SCREENSHOT HERE -->

```text
[ Genie Screenshot ]
```

---

## Pipeline Email Notification

<!-- ADD SCREENSHOT HERE -->

```text
[ Email Screenshot ]
```

---

## GitHub

### ADF Repository

<!-- ADD SCREENSHOT HERE -->

```text
[ ADF GitHub Repository Screenshot ]
```

### Databricks Repository

<!-- ADD SCREENSHOT HERE -->

```text
[ Databricks GitHub Repository Screenshot ]
```

### Portfolio Repository

<!-- ADD SCREENSHOT HERE -->

```text
[ Portfolio GitHub Repository Screenshot ]
```

---

# Project Repository Links

### Azure Data Factory Source

`medinexus-azure-data-factory`

### Azure Databricks Source

`medinexus-adb-repo`

### MediNexus Portfolio

`medinexus-azure-data-platform`

---

# Project Structure

The portfolio repository will progressively contain documentation and visual evidence.

```text
medinexus-azure-data-platform/
│
├── README.md
│
├── architecture/
│   ├── medinexus-architecture.png
│   └── data-flow.png
│
├── docs/
│   ├── architecture.md
│   ├── metadata-framework.md
│   ├── medallion-architecture.md
│   ├── data-quality.md
│   ├── monitoring.md
│   └── cicd.md
│
├── screenshots/
│   ├── azure/
│   ├── adf/
│   ├── databricks/
│   ├── dashboard/
│   ├── genie/
│   └── github/
│
└── README.md
```

---

# Key Engineering Concepts Demonstrated

This project demonstrates practical implementation of:

- Azure Data Factory
- Metadata-driven pipelines
- Parameterized pipelines
- Dynamic pipeline execution
- ForEach processing
- Lookup activities
- Execute Pipeline activities
- Conditional orchestration
- Incremental loading
- Watermark processing
- Full loading
- Append loading
- Merge/upsert processing
- ADLS Gen2
- Azure Databricks
- Apache Spark
- PySpark
- Delta Lake
- Unity Catalog
- Medallion Architecture
- Azure SQL metadata frameworks
- Audit logging
- Data quality validation
- Spark optimization
- Window functions
- Healthcare analytics
- Databricks Dashboards
- Databricks Genie
- Azure Key Vault
- Logic Apps
- GitHub source control
- CI/CD architecture

---

# Future Enhancements

The platform will continue to evolve.

Potential enhancements include:

- Complete automated CI/CD implementation
- Automated deployment of Databricks notebooks
- Automated ADF deployment
- Improved watermark management
- Late-arriving data handling
- More robust no-file handling
- Expanded data quality framework
- Automated test execution
- Additional healthcare analytical datasets
- More dashboard pages
- Improved data lineage documentation
- Environment separation such as Dev / Test / Production
- Automated infrastructure deployment
- Enhanced monitoring and alerting

---

# Lessons Learned

The project provides practical experience in designing a production-oriented Azure Data Engineering platform.

Key areas of learning include:

### Metadata-driven architecture

Centralized metadata can significantly reduce pipeline duplication and simplify onboarding of new datasets.

### Medallion Architecture

Separating raw, validated, and business-ready data improves organization and maintainability.

### Incremental processing

Watermark-based ingestion reduces unnecessary processing but requires careful handling of late-arriving and updated records.

### Delta Lake

Delta provides reliable transactional capabilities that are valuable for modern data lake architectures.

### Unity Catalog

Centralized governance becomes increasingly important as the number of datasets and users grows.

### Orchestration

Separating master, outer, and inner pipelines makes complex metadata-driven workflows easier to manage.

### Data validation

Successful pipeline execution does not necessarily mean the data is correct. Independent reconciliation of Gold metrics against Silver data is an important part of the implementation.

---

# Interview Highlights

MediNexus can be used to demonstrate practical understanding of the following interview topics:

## Azure Data Factory

- Metadata-driven pipelines
- Lookup
- ForEach
- Execute Pipeline
- Parameters
- Dynamic content
- Dependency conditions
- Incremental ingestion
- Watermark management
- Error handling
- Audit logging

## Databricks

- Spark architecture
- PySpark
- Delta Lake
- Medallion Architecture
- Unity Catalog
- Delta MERGE
- Structured transformations
- Window functions
- Spark optimization

## SQL

- CTEs
- Window functions
- Aggregations
- Conditional logic
- Metadata queries
- Audit queries
- Incremental filtering

## Azure

- ADLS Gen2
- Azure SQL Database
- Azure Key Vault
- Azure Data Factory
- Azure Databricks
- Logic Apps

## DevOps

- Git
- GitHub
- Feature branches
- Pull requests
- Repository separation
- CI/CD architecture

---

# Project Status

| Area | Status |
|---|---|
| Azure infrastructure | Completed |
| Source ingestion | Completed |
| Metadata framework | Completed |
| Landing layer | Completed |
| Bronze layer | Completed |
| Silver layer | Completed |
| Gold layer | Completed |
| Unity Catalog | Completed |
| Delta Lake implementation | Completed |
| Audit logging | Completed |
| Dashboard | Completed |
| Databricks Genie | Implemented |
| Email notification | Completed |
| GitHub source control | Implemented |
| Portfolio repository | Initial setup |
| Automated CI/CD | In progress |
| Advanced watermark handling | Enhancement in progress |

---

# Author

**Alvin David**

Azure Data Engineering Project

Focus areas:

- Azure Data Factory
- Azure Databricks
- PySpark
- Delta Lake
- Unity Catalog
- SQL
- Azure Data Engineering
- Metadata-Driven Architecture

---

## Disclaimer

This is a personal Azure Data Engineering portfolio project created for learning, engineering practice, and demonstration of modern data platform concepts.

The healthcare datasets used in the project are demonstration data and should not be interpreted as real patient information.
