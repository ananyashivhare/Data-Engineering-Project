# Credit Default Risk Analysis — End-to-End Azure Data Engineering & BI Project

## Overview
This project demonstrates a complete data engineering and analytics pipeline built on Microsoft Azure, leveraging Azure Data Factory, Azure Databricks, Azure Data Lake Storage (ADLS), and Tableau for business intelligence.  
The dataset used is the [UCI Default of Credit Card Clients Dataset](https://www.kaggle.com/datasets/uciml/default-of-credit-card-clients-dataset), which contains demographic and financial data of Taiwanese credit card clients. The goal is to analyze and visualize customer default patterns to support risk management and strategic decision-making.

## Project Objectives
- Design an end-to-end data pipeline for ingesting, transforming, and analyzing financial risk data.  
- Build KPI metrics that track customer default trends, utilization behavior, and demographic insights.  
- Develop Tableau dashboards for business stakeholders to monitor financial performance and risks.  
- Implement a scalable Azure architecture for future ML-based credit risk prediction.

 ## Tech Stack
| Layer | Tools & Services |
|-------|------------------|
| **Data Ingestion** | Azure Data Factory (ADF) |
| **Storage** | Azure Data Lake Storage Gen2 (ADLS) |
| **Processing** | Azure Databricks (PySpark, SQL) |
| **Transformation Layers** | Bronze → Silver → Gold |
| **Visualization** | Tableau Desktop / Tableau Public |
| **Orchestration** | ADF Pipelines + Databricks Jobs |
| **Languages** | SQL, Python (PySpark), Markdown |
| **Version Control** | Git & GitHub |

## Data Pipeline Architecture
[Architecture Diagram](architecture_pipeline_diagram.png)
Flow Summary:
1. Azure Data Factory (ADF) ingests the raw Kaggle CSV from local or web storage to the ADLS “raw” container.
2. Databricks Bronze layer cleans and standardizes schema and column names.
3. Silver layer performs data type casting, null handling, and feature derivation (e.g., average bills/payments).
4. Gold layer aggregates KPIs by demographic, age, and payment behavior for BI consumption.
5. Tableau Dashboard connects to Databricks SQL Warehouse to visualize risk KPIs in real time.
—--------------------------------------------------------------------------------------------

## Step-by-Step Project Workflow
1️⃣ Dataset & Storage Setup
- Download dataset: [Kaggle — Default of Credit Card Clients](https://www.kaggle.com/datasets/uciml/default-of-credit-card-clients-dataset)
- Upload the CSV (`default_of_credit_card_clients.csv`) into Azure Data Lake Storage Gen2 → `raw` container  
abfss://raw@.dfs.core.windows.net/default_of_credit_card_clients.csv
2️⃣ Data Ingestion — Azure Data Factory (ADF)
📄 Importable ADF Pipeline JSON  
[Download ingest_credit_default_pipeline.json](./ingest_credit_default_pipeline.json)
Pipeline Features
- Parameterized pipeline (Storage Account, Container, File Path)
- Activities:
- `GetMetadata` → Validate input file exists
- `CopyActivity` → Move raw file to ADLS
- `DatabricksNotebook` → Trigger Bronze ingest notebook
- `IfCondition` → Email notification if file missing

Usage
1. Open ADF → Author → Import pipeline JSON  
2. Update linked services and dataset paths  
3. Run in Debug mode to verify flow
ARM Template (Full Deployment)  
[Download adf_arm_template_ingest_pipeline.json](./adf_arm_template_ingest_pipeline.json)

3️⃣ Transformation — Azure Databricks
🔶 Bronze Layer
Load raw CSV → Apply schema → Write to `bronze_credit_default`
🔷 Silver Layer
Clean nulls, rename columns, convert datatypes
🟡 Gold Layer
Create KPI-level aggregated tables for Tableau

4️⃣ Business KPIs (Gold Layer Metrics)
| **KPI** | **Description** | **Formula** |
|----------|----------------|--------------|
| Default Rate (%) | % of customers defaulting | (Total Defaults / Total Customers) * 100 |
| Utilization Ratio | Average balance used vs credit limit | Avg Bill / Limit Balance |
| Payment Behavior | % paid vs billed | Avg Payment / Avg Bill |
| Demographic Risk | Default rate by gender, age, education | Grouped Aggregations |
| Credit Exposure | Avg credit limit by customer segment | AVG(Limit Balance) |

Gold Views Created
gold_default_overall
gold_default_by_age
gold_default_by_demo
gold_utilization_payment
gold_payment_ratio_buckets

5️⃣ Visualization — Tableau Dashboard
Connection:
Server: <Databricks SQL Warehouse Host>
HTTP Path: <Warehouse Path>
Authentication: Personal Access Token
Database: credit_risk

Tableau Dashboard Pages Overview
| **Page** | **Focus** | **Visuals** |
|-----------|------------|-------------|
| Executive Overview | Overall defaults, customer KPIs | KPI Cards, Trend Lines |
| Demographics | Age, gender, education risk | Heatmaps, Bars |
| Utilization Insights | Credit usage & defaults | Scatterplots, Histograms |
| Payment Behavior | Repayment ratio analysis | Donut, Bar Charts |
| Risk Segmentation | Predictive/score-based | Risk Bucket Bars, Tables |

Sample KPI SQL
SELECT education, COUNT() AS customers,
SUM(default_flag) AS defaults,
ROUND(100.0SUM(default_flag)/COUNT(), 2) AS default_rate_pct
FROM credit_risk.silver_credit_default
GROUP BY education;
Sample Tableau Calculated Field
Default Rate (%) = SUM([defaults]) / SUM([customers])  100

6️⃣ Databricks Jobs & Cluster Recommendations
📄 Databricks Jobs + Cluster Configuration Guide
Recommended Cluster Settings
Worker Type: Standard_DS3_v2
Min Workers: 2 | Max Workers: 8
Auto-termination: 120 mins
Runtime: Databricks Runtime 14.x (includes Delta Engine)
Libraries: pandas, pyspark, delta, matplotlib

## KPI Dashboard Mockup (Tableau Layout)
Executive KPIs: Default Rate | Total Customers | Avg Limit 
Trend: Monthly Default %
Heatmap: Default by Age & Education
Scatter: Utilization vs Payment Ratio

## Pipeline Orchestration Flow
| Step | Component | Description |
| ---- | ------------------- | ------------------------ | 
| 1 | ADF | Ingest raw CSV → ADLS | 
| 2 | Databricks (Bronze) | Basic cleaning, schema | 
| 3 | Databricks (Silver) | Feature engineering | 
| 4 | Databricks (Gold) | KPI aggregations | 
| 5 | Tableau | Visualization & BI layer |

## Future Enhancements
Add ML model (logistic regression / XGBoost) to predict default risk.
Automate Tableau extract refresh post Databricks job completion.
Integrate with Azure Synapse Analytics for enterprise-scale analytics.
Build CI/CD pipeline for data jobs using Azure DevOps or GitHub Actions.

## Key Learnings and aspects
Building modular ETL pipelines with ADF + Databricks.
Implementing Bronze–Silver–Gold data architecture.
Designing business-ready KPIs from raw data.
Developing interactive BI dashboards in Tableau.
Applying financial domain analytics for risk monitoring.

## Repository Structure
├── architecture_pipeline_diagram.png
├── ingest_credit_default_pipeline.json
├── adf_arm_template_ingest_pipeline.json
├── databricks_jobs_and_cluster_recommendations.md
├── bronze_to_gold_notebooks/
│   ├── bronze_ingest.ipynb
│   ├── silver_cleaning.ipynb
│   ├── gold_kpis.ipynb
├── tableau/
│   ├── Credit_Risk_Dashboard.twbx
│   ├── dashboard_screenshots/
└── README.md


## References
UCI Credit Card Dataset on Kaggle
Azure Databricks Documentation
Azure Data Factory Documentation
Tableau Documentation

