# RetailMart — End-to-End Data Engineering Platform

## 📌 Project Overview

**RetailMart Data Platform** is an end-to-end cloud data engineering project designed to simulate a real-world retail analytics environment.

The platform ingests data from multiple sources, including PostgreSQL databases, CSV files, and REST APIs. Azure Data Factory is used for metadata-driven orchestration and incremental data ingestion. Azure Data Lake Storage Gen2 acts as the central data lake, while Azure Databricks, PySpark, Delta Lake, Unity Catalog, and Lakeflow Declarative Pipelines are used for data transformation, data quality, governance, and analytics.

The curated business data is ultimately made available in Snowflake for analytical workloads and reporting.

### Business Objective

The platform enables business teams to analyze:

* Daily and monthly sales
* Product performance
* Customer purchasing behavior
* Store performance
* Inventory levels
* Product returns
* Revenue trends
* Customer value
* Sales by channel
* Data quality and pipeline health

---

# 🏗️ Architecture

```text
                       DATA SOURCES
                            │
              ┌─────────────┼─────────────┐
              │             │             │
              ▼             ▼             ▼
         PostgreSQL      CSV Files      REST API
              │             │             │
              └─────────────┼─────────────┘
                            │
                            ▼
                  ┌───────────────────┐
                  │   Azure Data      │
                  │     Factory       │
                  │                   │
                  │ Metadata-Driven   │
                  │ Orchestration     │
                  └─────────┬─────────┘
                            │
                            ▼
                  ┌───────────────────┐
                  │   ADLS Gen2       │
                  │                   │
                  │     RAW           │
                  └─────────┬─────────┘
                            │
                            ▼
                  ┌───────────────────┐
                  │ Azure Databricks  │
                  │                   │
                  │     PySpark       │
                  │   Delta Lake      │
                  └─────────┬─────────┘
                            │
                  ┌─────────┴─────────┐
                  │                   │
                  ▼                   ▼
             Bronze Delta        Silver Delta
                  │                   │
                  └─────────┬─────────┘
                            │
                            ▼
               Lakeflow Declarative
                    Pipelines
                            │
                            ▼
                       Gold Delta
                            │
                  ┌─────────┴─────────┐
                  │                   │
                  ▼                   ▼
             Unity Catalog       Snowflake
              Governance          Analytics
                                      │
                                      ▼
                                  Power BI
```

---

# ☁️ Technology Stack

| Technology                         | Purpose                                       |
| ---------------------------------- | --------------------------------------------- |
| **Azure Data Factory**             | Data ingestion and pipeline orchestration     |
| **Azure Data Lake Storage Gen2**   | Central cloud data lake                       |
| **Azure Databricks**               | Distributed data processing                   |
| **PySpark**                        | Large-scale data transformation               |
| **Delta Lake**                     | Reliable lakehouse storage                    |
| **Unity Catalog**                  | Data governance, access control and discovery |
| **Lakeflow Declarative Pipelines** | Declarative data pipelines and data quality   |
| **Snowflake**                      | Cloud data warehouse and analytics            |
| **PostgreSQL**                     | Transactional source system                   |
| **REST API**                       | External data source                          |
| **CSV**                            | External file-based data source               |
| **SQL**                            | Data transformation and analytics             |
| **Power BI**                       | Business reporting and visualization          |
| **GitHub**                         | Source control and project documentation      |

---

# 🏢 Business Scenario

RetailMart operates multiple retail stores and an online sales channel.

The company generates data from:

### Customer Management

```text
Customer
    │
    ├── Customer ID
    ├── Name
    ├── Email
    ├── Location
    ├── Customer Segment
    └── Signup Date
```

### Product Management

```text
Product
    │
    ├── Product ID
    ├── Product Name
    ├── Category
    ├── Brand
    ├── Selling Price
    ├── Cost Price
    └── Supplier
```

### Sales

```text
Order
    │
    ├── Order ID
    ├── Customer
    ├── Store
    ├── Order Date
    ├── Channel
    ├── Payment Method
    └── Status
```

### Inventory

```text
Inventory
    │
    ├── Store
    ├── Product
    ├── Stock Quantity
    └── Reorder Level
```

### Returns

```text
Returns
    │
    ├── Order Item
    ├── Return Date
    ├── Return Quantity
    ├── Return Reason
    └── Refund Amount
```

---

# 📂 Source Database

The initial transactional source is PostgreSQL.

### Tables

```text
retailmart
│
├── customers
├── products
├── stores
├── inventory
├── orders
├── order_items
└── returns
```

The database also contains ETL control tables:

```text
├── etl_config
└── etl_watermark
```

These tables support metadata-driven and incremental data ingestion.

---

# 🔄 Data Ingestion

Azure Data Factory will implement a **metadata-driven ingestion framework**.

Instead of creating a separate pipeline for every source table, ADF reads the configuration from:

```text
etl_config
```

Example:

```text
source_table     load_type       target_folder
------------------------------------------------
customers        incremental     raw/customers
products         incremental     raw/products
orders           incremental     raw/orders
order_items      incremental     raw/order_items
stores           incremental     raw/stores
inventory        incremental     raw/inventory
returns          incremental     raw/returns
```

ADF then dynamically processes the configured tables.

### High-Level Flow

```text
Lookup
   │
   ▼
Filter Active Tables
   │
   ▼
ForEach
   │
   ▼
Dynamic Copy Activity
   │
   ▼
PostgreSQL → ADLS Gen2
```

---

# 🔁 Incremental Data Loading

The project implements incremental ingestion using a watermark-based approach.

The control table:

```text
etl_watermark
```

stores the last successfully processed timestamp for each source table.

Example:

```text
source_table       watermark_value
-------------------------------------------
customers          2026-09-17 10:30:00
orders             2026-09-17 11:45:00
products           2026-09-17 09:20:00
```

ADF uses the watermark to dynamically generate queries such as:

```sql
SELECT *
FROM orders
WHERE updated_at > :last_watermark
```

This prevents unnecessary full extraction of large source tables.

---

# 🗄️ Data Lake Structure

ADLS Gen2 will contain the following structure:

```text
retail-data/
│
├── raw/
│   ├── customers/
│   ├── products/
│   ├── stores/
│   ├── inventory/
│   ├── orders/
│   ├── order_items/
│   ├── returns/
│   └── exchange_rates/
│
├── processed/
│
└── archive/
```

---

# 🥉 Bronze Layer

The Bronze layer stores source data in Delta format with minimal transformation.

Example:

```text
retail_catalog.bronze.customers
retail_catalog.bronze.products
retail_catalog.bronze.orders
retail_catalog.bronze.order_items
retail_catalog.bronze.inventory
retail_catalog.bronze.returns
```

Purpose:

* Preserve source data
* Enable reprocessing
* Maintain historical records
* Provide an auditable ingestion layer

---

# 🥈 Silver Layer

The Silver layer contains cleaned and standardized business data.

Typical transformations include:

* Removing duplicate records
* Handling null values
* Standardizing data types
* Standardizing dates and timestamps
* Validating relationships
* Cleaning customer information
* Validating product prices
* Calculating derived fields

Example:

```text
retail_catalog.silver.customers
retail_catalog.silver.products
retail_catalog.silver.orders
retail_catalog.silver.order_items
retail_catalog.silver.inventory
```

---

# 🥇 Gold Layer

The Gold layer contains business-ready analytical datasets.

Example:

```text
retail_catalog.gold.daily_sales
retail_catalog.gold.customer_360
retail_catalog.gold.product_performance
retail_catalog.gold.store_performance
retail_catalog.gold.monthly_revenue
retail_catalog.gold.return_analysis
```

These tables are optimized for analytical workloads and reporting.

---

# 🧱 Unity Catalog

Unity Catalog is used as the centralized governance layer for the Databricks environment.

Logical structure:

```text
retail_catalog
│
├── bronze
│
├── silver
│
└── gold
```

Access can be controlled based on user responsibilities.

Example:

```text
Data Engineer
    │
    ├── Bronze
    ├── Silver
    └── Gold

Data Analyst
    │
    └── Gold
```

This project also demonstrates concepts such as:

* Catalogs
* Schemas
* Managed tables
* External locations
* Storage credentials
* Permissions
* Data discovery
* Access control

---

# ⚙️ Lakeflow Declarative Pipelines

Lakeflow Declarative Pipelines are used for declarative data transformation and data quality.

Example pipeline:

```text
Bronze
   │
   ▼
Silver
   │
   ▼
Gold
```

Data quality expectations will validate conditions such as:

```text
order_id IS NOT NULL
customer_id IS NOT NULL
quantity > 0
unit_price >= 0
```

Invalid records can be handled according to the defined data-quality rules.

---

# ❄️ Snowflake Integration

Curated Gold data is loaded into Snowflake for analytical workloads.

Example structure:

```text
RETAIL_DB
│
├── RAW
│
├── STAGING
│
└── ANALYTICS
    │
    ├── CUSTOMER_360
    ├── DAILY_SALES
    ├── PRODUCT_PERFORMANCE
    ├── STORE_PERFORMANCE
    └── MONTHLY_REVENUE
```

Snowflake is used for:

* Analytical SQL
* Aggregations
* Dimensional modeling
* Business reporting
* Historical analysis
* BI consumption

---

# 📊 Business Analytics

The final platform supports analysis such as:

### Sales Analysis

```text
Daily Revenue
Monthly Revenue
Revenue by Store
Revenue by Product
Revenue by Category
Revenue by Channel
```

### Customer Analytics

```text
Total Customer Spending
Number of Orders
Average Order Value
Customer Segmentation
Customer Lifetime Value
```

### Product Analytics

```text
Top Selling Products
Low Performing Products
Units Sold
Revenue by Product
Profit Margin
```

### Inventory Analytics

```text
Current Stock
Low Stock Products
Reorder Requirements
Inventory by Store
```

### Returns Analytics

```text
Return Rate
Return Quantity
Refund Amount
Return Reasons
Returns by Product
```

---

# 🔍 Data Quality Framework

The project includes data quality checks such as:

```text
✓ Null validation
✓ Duplicate detection
✓ Referential integrity
✓ Invalid price detection
✓ Invalid quantity detection
✓ Missing source file detection
✓ Record count validation
✓ Incremental load validation
```

Example:

```text
orders
   │
   ├── order_id NOT NULL
   ├── customer_id NOT NULL
   ├── valid order status
   └── valid order date
```

---

# 🔐 Security & Governance

The project demonstrates:

* Azure Managed Identity
* Azure Key Vault integration
* Unity Catalog permissions
* ADLS access control
* Secret management
* Role-based access
* Separation of data layers

Credentials and secrets are **not stored in GitHub**.

---

# 🚨 Error Handling & Monitoring

ADF pipelines will include:

```text
Success
   │
   ▼
Validation
   │
   ├── Pass → Continue
   │
   └── Fail → Error Handling
```

The implementation will include:

* Activity-level retries
* Failure paths
* Pipeline monitoring
* Data validation
* Logging
* Pipeline run tracking
* Incremental load monitoring

---

# 🔄 End-to-End Pipeline

The final workflow will look like:

```text
1. Source Systems
       ↓
2. ADF Master Pipeline
       ↓
3. Lookup Configuration
       ↓
4. Filter Active Sources
       ↓
5. ForEach Source
       ↓
6. Incremental Extraction
       ↓
7. ADLS Raw
       ↓
8. Databricks
       ↓
9. Bronze Delta
       ↓
10. Data Quality
       ↓
11. Silver Delta
       ↓
12. Gold Delta
       ↓
13. Snowflake
       ↓
14. Power BI
```

---

# 📁 Project Structure

```text
retailmart-data-platform/
│
├── README.md
│
├── sql/
│   ├── 01_create_tables.sql
│   ├── 02_insert_data.sql
│   ├── 03_control_tables.sql
│   └── 04_test_queries.sql
│
├── adf/
│   ├── pipelines/
│   ├── datasets/
│   ├── linked-services/
│   └── triggers/
│
├── databricks/
│   ├── bronze/
│   ├── silver/
│   ├── gold/
│   └── lakeflow/
│
├── snowflake/
│   ├── database/
│   ├── schemas/
│   ├── tables/
│   └── procedures/
│
├── data/
│   └── sample/
│
├── docs/
│   ├── architecture/
│   ├── data-model/
│   └── screenshots/
│
└── powerbi/
    └── dashboards/
```

---

# 🎯 Key Engineering Concepts Demonstrated

This project demonstrates practical experience with:

### Azure Data Factory

* Metadata-driven pipelines
* Parameterization
* Dynamic expressions
* Incremental loading
* Watermark framework
* Control tables
* ForEach processing
* Lookup activity
* Get Metadata
* Filter
* If Condition
* Variables
* Web Activity
* Execute Pipeline
* Until
* Validation
* Databricks integration
* Snowflake integration
* Pipeline monitoring
* Error handling

### Azure Databricks

* Workspace
* Compute
* PySpark
* Delta Lake
* Bronze/Silver/Gold architecture
* Data transformations
* Data quality
* Lakeflow Declarative Pipelines

### Unity Catalog

* Catalogs
* Schemas
* Tables
* Storage
* External locations
* Access control
* Data governance

### Snowflake

* Database and schema design
* Analytical tables
* SQL transformations
* Incremental processing
* Dimensional modeling
* Data warehouse concepts

--

# 🚀 Project Status

| Component                | Status         |
| ------------------------ | -------------- |
| PostgreSQL source        | 🟢 Completed   |
| Source tables            | 🟢 Completed   |
| ETL control tables       | 🟢 Completed   |
| ADLS Gen2                | 🔄 In Progress |
| ADF ingestion            | 🔄 Planned     |
| Metadata-driven pipeline | 🔄 Planned     |
| Incremental loading      | 🔄 Planned     |
| Databricks Bronze        | 🔄 Planned     |
| Databricks Silver        | 🔄 Planned     |
| Unity Catalog            | 🔄 Planned     |
| Lakeflow / DLT           | 🔄 Planned     |
| Gold layer               | 🔄 Planned     |
| Snowflake                | 🔄 Planned     |
| Data quality             | 🔄 Planned     |
| Monitoring               | 🔄 Planned     |
| Power BI                 | 🔄 Planned     |

---

# 💡 Future Enhancements

Possible future improvements include:

* Implement SCD Type 2 for customer/product dimensions
* Add automated data-quality reporting
* Add pipeline audit tables
* Add CI/CD
* Add Dev/Test/Prod environments
* Implement automated deployment
* Add streaming data ingestion
* Add Kafka integration
* Add machine-learning features
* Build customer churn prediction
* Build product demand forecasting

---

# 👨‍💻 Author

**Kiyasudeen Jamal M.**

Data Engineer | Azure | Databricks | Snowflake | PySpark | SQL | Python

---

## ⚠️ Disclaimer

This is a personal hands-on data engineering project created for learning, portfolio development, and demonstrating cloud data engineering concepts.

No confidential company data or credentials are included in this repository.
