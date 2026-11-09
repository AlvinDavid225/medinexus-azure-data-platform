# MediNexus — Azure Healthcare Data Platform

MediNexus is a portfolio project demonstrating an Azure data engineering workflow for healthcare demonstration data. It brings relational and file-based sources through Azure Data Factory (ADF) into a metadata-driven Landing, Bronze, Silver, and Gold flow, with Databricks transformations and analytical consumption.

> **Data notice:** The datasets are for demonstration and do not represent real patient information. This is a portfolio implementation, not a production healthcare system.

## Project overview

- **Sources:** Neon PostgreSQL and CSV files in Azure Data Lake Storage Gen2 (ADLS Gen2)
- **Orchestration and ingestion:** Azure Data Factory
- **Control metadata and audit:** Azure SQL Database
- **Processing and storage:** Azure Databricks, PySpark, Delta Lake, and Unity Catalog
- **Consumption:** Databricks dashboards and Genie
- **Credentials:** Azure Key Vault

## Architecture

ADF reads dataset configuration and coordinates ingestion and processing. Source data is staged in Landing, loaded into Bronze, transformed into Silver, and shaped into Gold datasets for analytics. Azure SQL holds pipeline configuration, audit records, and watermarks; Databricks handles the lakehouse transformations.

```text
Neon PostgreSQL ─┐
                 ├─> ADF ─> ADLS Landing ─> Databricks Bronze ─> Silver ─> Gold ─> Dashboard / Genie
ADLS CSV files ──┘           Azure SQL control metadata ────────────────┘
```

## Data sources

The current configuration covers five datasets:

| Source | Dataset | Load mode | Incremental field | Merge key |
|---|---|---|---|---|
| Neon PostgreSQL | `hospitals` | Full | — | — |
| Neon PostgreSQL | `doctors` | Full | — | — |
| Neon PostgreSQL | `patients` | Merge | `updated_at` | `patient_id` |
| ADLS CSV | `lab_results` | Append | `reported_at` | — |
| ADLS CSV | `insurance_providers` | Full | — | — |

## Azure Data Factory

ADF coordinates the configured source-to-Silver workflow. The master pipeline selects work from metadata and invokes parameterized processing pipelines; source-specific activities copy PostgreSQL data as Parquet and CSV files into timestamped Landing paths. Databricks notebooks then load Landing to Bronze and Bronze to Silver. A separate Silver-to-Gold workflow produces the analytical datasets.

The orchestration also includes a Logic App email notification step. ADF execution status and table-level processing details are recorded in Azure SQL for operational review.

![ADF notification pipeline](./screenshots/azure-data-factory/PL_SEND_EMAIL.png)

## Metadata framework and audit

Azure SQL separates control data from analytical data. The framework uses:

- `ctrl.table_config` to define source, dataset, load mode, watermark field, and merge key settings.
- `ctrl.watermark` to store incremental processing state for configured datasets.
- `ctrl.audit_log` to track ADF run, table, processing stage, status, record counts, and execution timestamps.

This configuration drives the current Full, Append, and Merge ingestion patterns. Incremental state is used for configured incremental sources; watermark edge cases remain an enhancement area.

![Successful table-level audit records](./screenshots/azure-sql-metadata/sql-audit-log-success.png)

## Azure Databricks

Databricks runs parameterized PySpark notebooks over Delta tables. Unity Catalog organizes the platform in the `adb_medinexus` catalog:

```text
adb_medinexus
├── landing  (external volume: landing_volume)
├── bronze
├── silver
└── gold
```

The current Gold datasets are `patient_360`, `hospital_daily_summary`, and `lab_test_trends`. Silver processing supports the configured Full, Append, and Merge modes; the Merge notebook validates that configured keys exist in its input before applying Delta merge behavior.

![Unity Catalog structure](./screenshots/databricks/unity-catalog/databricks-catalog-structure.png)

## Validation

Validation and reconciliation have been performed manually after pipeline runs. Checks included null and duplicate keys, source-to-Bronze and Bronze-to-Silver counts, Silver-to-Gold reconciliation, and independent checks of selected Gold metrics such as laboratory totals, patient counts, active doctors, and trends.

One recorded run contained the following row counts:

| Layer / dataset | Rows |
|---|---:|
| Bronze and Silver `hospitals` | 50 each |
| Bronze and Silver `doctors` | 500 each |
| Bronze and Silver `patients` | 5,000 each |
| Bronze and Silver `lab_results` | 20 each |
| Bronze and Silver `insurance_providers` | 30 each |
| Gold `patient_360` | 5,000 |
| Gold `hospital_daily_summary` | 20 |
| Gold `lab_test_trends` | 8 |

Gold outputs have different analytical grains, so their counts are not expected to equal the source-layer counts.

## Analytics

The Databricks dashboard uses Gold datasets and includes Executive Healthcare Overview, Patient Analytics, Hospital Performance, and Laboratory Analytics views. Databricks Genie is also configured for natural-language questions against curated Gold data.

![Executive healthcare dashboard](./screenshots/dashboards/dashboard-executive-overview.png)

## Security

Azure Key Vault stores development connection secrets for Databricks, Azure SQL Database, and PostgreSQL. Unity Catalog organizes Databricks data assets; it is documented with the Databricks architecture above.

![Azure Key Vault overview](./screenshots/azure-infrastructure/Key%20Vault.png)

## GitHub and CI/CD

Source control is separated by responsibility across `medinexus-azure-data-factory`, `medinexus-adb-repo`, and the portfolio repository `medinexus-azure-data-platform`. The portfolio repository is intended for project documentation and evidence; it is distinct from the ADF and Databricks source repositories.

Git-based source control is established. Automated deployment for ADF and Databricks is **in progress**; the feature branch, pull request, review, and deployment flow is the planned direction, not a claim of completed CI/CD.

## Project status

| Area | Status |
|---|---|
| Source ingestion and metadata-driven processing | Implemented |
| Landing, Bronze, Silver, and Gold flow | Implemented |
| Unity Catalog and Delta tables | Implemented |
| Azure SQL audit and watermark tracking | Implemented |
| Dashboard, Genie, and Logic App notification | Implemented |
| GitHub source control | Established |
| Automated CI/CD | In progress |

## Future enhancements

- Complete and document automated ADF and Databricks deployment.
- Extend watermark handling for late-arriving data and recovery scenarios.
- Automate data quality checks and test execution.
- Add environment separation and infrastructure deployment automation.
- Expand monitoring, lineage documentation, datasets, and analytical views.

## Evidence

Additional screenshots cover the Landing volume and Bronze, Silver, and Gold tables under `screenshots/databricks/unity-catalog/`; dashboard detail views and Genie examples are under `screenshots/dashboards/` and `screenshots/databricks/databricks-genie/`. Key Vault secrets are shown in `screenshots/azure-infrastructure/key-vault-secrets.png`.

## Author

**Alvin David** · Azure Data Engineering portfolio project
