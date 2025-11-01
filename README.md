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
#### 🔶 Bronze Layer - Load raw CSV → Apply schema → Write to `bronze_credit_default`
#### 🔷 Silver Layer - Clean nulls, rename columns, convert datatypes
#### 🟡 Gold Layer - Create KPI-level aggregated tables for Tableau

4️⃣ Business KPIs (Gold Layer Metrics)

### KPI Calculations

| KPI | Description | Formula |
|-----|-------------|---------|
| **Default Rate (%)** | Percentage of customers defaulting | `(Total Defaults / Total Customers) * 100` |
| **Utilization Ratio** | Average balance used vs credit limit | `(Avg Bill / Limit Balance) * 100` |
| **Credit Exposure** | Average credit limit by customer segment | `AVG(Limit Balance)` |
| **Demographic Risk** | Default rate by gender, age, education | Grouped Aggregations |
| **Payment Status Risk** | Default rate by payment behavior | Grouped by PAY_0 status |

### Gold Layer Tables Created

The following tables are created in the `credit_risk` database and are visible in Databricks Catalog:

#### 1. **gold_customer_agg** - Customer-Level Aggregations
Complete customer profile with all financial metrics.

**Columns:**
- `ID` - Customer identifier
- `LIMIT_BAL` - Credit limit
- `SEX`, `EDUCATION`, `MARRIAGE`, `AGE` - Demographics
- `latest_pay_status` - Most recent payment status
- `avg_pay_status` - Average payment status across months
- `avg_monthly_bill` - Average monthly bill amount
- `max_monthly_bill` - Maximum monthly bill
- `min_monthly_bill` - Minimum monthly bill
- `avg_bill_6months` - 6-month average bill
- `avg_monthly_payment` - Average monthly payment
- `max_monthly_payment` - Maximum payment made
- `min_monthly_payment` - Minimum payment made
- `utilization_ratio` - Credit utilization percentage
- `default_flag` - Default indicator (Y/N)
- `record_count` - Number of records

**Use Case:** Individual customer analysis, customer segmentation, personalized risk scoring

#### 2. **gold_risk_segments** - Risk Segment Analysis
Combines spending patterns with payment status to identify high-risk segments.

**Columns:**
- `spending_segment` - High/Medium/Low Spender (based on avg_bill_amt)
- `payment_status_segment` - Good Standing/Revolving/Delinquent
- `default_flag` - Default indicator
- `customer_count` - Number of customers in segment
- `avg_credit_limit` - Average credit limit for segment
- `avg_bill_amount` - Average bill for segment
- `avg_age` - Average customer age
- `default_rate_pct` - Default percentage for segment

**Use Case:** Risk assessment, targeted interventions, portfolio management

#### 3. **gold_education_agg** - Education Demographics
Default patterns by education level.

**Columns:**
- `EDUCATION` - Education level (0=Unknown, 1=Graduate, 2=University, 3=High School, 4=Others)
- `customer_count` - Number of customers
- `avg_credit_limit` - Average credit limit
- `avg_bill_amount` - Average bill amount
- `avg_age` - Average age
- `default_rate_pct` - Default percentage

**Use Case:** Demographic analysis, education-based risk profiling

#### 4. **gold_age_group_agg** - Age Group Analysis
Default patterns across different age groups.

**Columns:**
- `age_group` - Age bracket (18-24, 25-34, 35-44, 45-54, 55+)
- `default_flag` - Default indicator
- `customer_count` - Number of customers
- `avg_credit_limit` - Average credit limit
- `avg_bill_amount` - Average bill amount
- `avg_age` - Average age in group

**Use Case:** Age-based risk assessment, lifecycle marketing

#### 5. **gold_gender_agg** - Gender Analysis
Default and financial patterns by gender.

**Columns:**
- `gender` - Gender (M/F)
- `default_flag` - Default indicator
- `customer_count` - Number of customers
- `avg_credit_limit` - Average credit limit
- `avg_bill_amount` - Average bill amount
- `avg_age` - Average age

**Use Case:** Gender-based insights, demographic segmentation

#### 6. **gold_scorecard** - Default Prediction Scorecard
Matrix of credit utilization vs payment status for risk prediction.

**Columns:**
- `credit_utilization` - High/Medium/Low Utilization
- `latest_payment_status` - On-Time/1 Month Late/2+ Months Late/Revolving
- `default_flag` - Default indicator
- `count` - Number of customers
- `avg_age` - Average age
- `avg_limit` - Average credit limit
- `avg_bill` - Average bill amount

**Use Case:** Predictive modeling, risk scoring, early warning system

---

### Sample KPI Queries

#### Overall Default Rate
```sql
SELECT 
    COUNT(*) AS total_customers,
    SUM(CASE WHEN default_flag = 'Y' THEN 1 ELSE 0 END) AS total_defaults,
    ROUND(100.0 * SUM(CASE WHEN default_flag = 'Y' THEN 1 ELSE 0 END) / COUNT(*), 2) AS default_rate_pct
FROM credit_risk.gold_customer_agg;
```

#### Default Rate by Education
```sql
SELECT 
    EDUCATION,
    customer_count AS customers,
    ROUND(default_rate_pct, 2) AS default_rate_pct,
    avg_credit_limit,
    avg_bill_amount
FROM credit_risk.gold_education_agg
ORDER BY default_rate_pct DESC;
```

#### High-Risk Segments
```sql
SELECT 
    spending_segment,
    payment_status_segment,
    customer_count,
    default_rate_pct
FROM credit_risk.gold_risk_segments
WHERE default_rate_pct > 0
ORDER BY default_rate_pct DESC
LIMIT 10;
```

#### Credit Utilization Analysis
```sql
SELECT 
    credit_utilization,
    latest_payment_status,
    default_flag,
    count AS customer_count,
    avg_limit,
    avg_bill
FROM credit_risk.gold_scorecard
ORDER BY count DESC;
```

---

### Tableau Dashboard Connection

**Connection Details:**
- **Server:** `<your-databricks-workspace>.azuredatabricks.net`
- **HTTP Path:** `/sql/1.0/warehouses/<warehouse-id>`
- **Authentication:** Personal Access Token
- **Database:** `credit_risk`

**Available Tables for Visualization:**
1. `gold_customer_agg` - Individual customer metrics
2. `gold_risk_segments` - Segment-level risk analysis
3. `gold_education_agg` - Education demographics
4. `gold_age_group_agg` - Age group insights
5. `gold_gender_agg` - Gender breakdown
6. `gold_scorecard` - Utilization × Payment matrix

**Sample Tableau Calculated Fields:**

```
// Default Rate (%)
SUM([defaults]) / SUM([customers]) * 100

// Utilization Rate
AVG([utilization_ratio])

// High Risk Flag
IF [default_rate_pct] > 30 THEN "High Risk"
ELSEIF [default_rate_pct] > 15 THEN "Medium Risk"
ELSE "Low Risk"
END
```

---

### Key Insights from Gold Layer

Based on the aggregated data:

1. **Risk Segmentation:** Customers with high utilization (>80%) and delinquent payment status show significantly higher default rates
2. **Age Patterns:** Younger age groups (18-34) may show different default patterns than older groups
3. **Education Impact:** Default rates vary across education levels, useful for risk profiling
4. **Payment Behavior:** Payment status (PAY_0) is a strong indicator of default risk
5. **Spending Patterns:** High spenders with poor payment records are the highest risk segment

---

5️⃣ Visualization — Tableau Dashboard

### Dashboard Pages

| Page | Focus | Visuals | Data Source |
|------|-------|---------|-------------|
| **Executive Overview** | Overall defaults, KPIs | KPI Cards, Trend Lines | `gold_customer_agg` |
| **Risk Segments** | Spending vs Payment behavior | Heatmap, Bar Charts | `gold_risk_segments` |
| **Demographics** | Age, gender, education risk | Stacked Bars, Heatmaps | `gold_education_agg`, `gold_age_group_agg`, `gold_gender_agg` |
| **Utilization Analysis** | Credit usage patterns | Scatter Plot, Histogram | `gold_scorecard` |
| **Customer Details** | Individual customer drill-down | Table, Detail Cards | `gold_customer_agg` |

### Visuals

**Executive KPIs:**
- Default Rate % (Big Number)
- Total Customers (Big Number)
- Avg Credit Limit (Big Number)
- Default Trend over Time (Line Chart)

**Risk Heatmap:**
- X-Axis: Credit Utilization (Low/Medium/High)
- Y-Axis: Payment Status (On-Time/Late/Delinquent)
- Color: Default Rate %
- Source: `gold_scorecard`

**Demographics:**
- Bar Chart: Default Rate by Age Group (`gold_age_group_agg`)
- Bar Chart: Default Rate by Education (`gold_education_agg`)
- Pie Chart: Gender Distribution (`gold_gender_agg`)

**Risk Segments:**
- Stacked Bar: Spending Segment with Payment Status breakdown
- Table: Top 10 High-Risk Segments
- Source: `gold_risk_segments`

---

## Accessing Gold Tables

### In Databricks

1. **Catalog Explorer:**
   - Click "Catalog" in left sidebar
   - Navigate to `credit_risk` database
   - View all 6 gold tables with schema and data preview

2. **SQL Editor:**
   ```sql
   -- List all tables
   SHOW TABLES IN credit_risk;
   
   -- Query any table
   SELECT * FROM credit_risk.gold_risk_segments;
   ```

3. **Python Notebook:**
   ```python
   # Read any gold table
   df = spark.table("credit_risk.gold_customer_agg")
   display(df)
   
   # Or read from path
   df = spark.read.format("delta").load("/mnt/cc/gold/credit_default/customer_agg")
   ```

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
