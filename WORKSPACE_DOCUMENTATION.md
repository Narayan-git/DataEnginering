# DataEngineering Workspace Documentation

**Last Updated:** December 2, 2025  
**Repository:** DataEnginering  
**Owner:** Narayan-git  
**Branch:** main

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [Workspace Structure](#workspace-structure)
3. [Architecture & Technology Stack](#architecture--technology-stack)
4. [Module Breakdown](#module-breakdown)
5. [Data Pipeline Flow](#data-pipeline-flow)
6. [Key Concepts](#key-concepts)
7. [Implementation Details](#implementation-details)

---

## 🎯 Project Overview

### Purpose
This is a **Databricks Lakehouse** project implementing a unified, scalable data pipeline for SportsBar order details. The project aims to integrate SportsBar's order data from diverse, unstructured sources (spreadsheets, cloud drives, APIs) into a structured, reliable data warehouse for the FMCG (Fast-Moving Consumer Goods) data analytics team.

### Primary Objective
Create a **single, reliable source** for product sales tracking and cross-company planning using the Medallion Architecture (Bronze → Silver → Gold layers).

### Target Use Case
- SportsBar order details tracking
- FMCG data analytics and reporting
- Product performance analysis
- Sales forecasting

---

## 📁 Workspace Structure

```
DataEnginering/
├── README.md                           # Root documentation
├── WORKSPACE_DOCUMENTATION.md          # This file
├── Databricks_project_1/              # Main project folder
│   ├── README.MD                      # Project planning & architecture
│   ├── 1_setup_catalog/               # Catalog initialization & utilities
│   │   ├── .setup.py                  # Setup script
│   │   ├── utilities.py               # Shared schema definitions
│   │   └── dim_date_table_creation.py # Date dimension creation
│   ├── 2_dimension_data_processing/   # Dimension table ETL
│   │   ├── 1_customer_data_processing.py      # Customer dimension
│   │   ├── 2_products_data_processing.py      # Product dimension
│   │   └── 3_pricing_data_processing.py       # Pricing dimension
│   └── 3_fact_data_processing/        # Fact table ETL
│       ├── 1_full_load_fact.py        # Full load of orders
│       └── 2_incremental_load_fact.py # Incremental load of orders
├── factory/                           # Azure Data Factory configs
│   └── nsahu-de-adf.json
├── integrationRuntime/                # Integration Runtime config
│   └── AutoResolveIntegrationRuntime.json
├── managedVirtualNetwork/             # Virtual Network config
│   └── default.json
└── Spark/                             # Spark setup documentation
    └── PreReqInstallation/
        ├── InstallationSteps.txt
        └── SparkOperation.txt
```

---

## 🏗️ Architecture & Technology Stack

### Platform & Storage
| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Compute Platform** | Databricks (Free Edition) | Serverless compute, unified storage & processing |
| **Storage Layer** | Azure Data Lake Storage Gen2 (ADLS Gen2) | Cloud-native data storage |
| **Data Format** | Delta Lake | ACID compliance, schema enforcement, efficient incremental updates |
| **Processing Engine** | Apache Spark (Python/SQL) | Distributed data processing |
| **Orchestration** | Databricks Workflows | Scheduling & daily load management |

### Data Architecture Pattern
**Medallion Architecture** (3-layer model):
- **Bronze Layer:** Raw ingestion with minimal transformation
- **Silver Layer:** Cleaned, conformed, and deduplicated data
- **Gold Layer:** Business-ready, aggregated analytics data

---

## 📊 Module Breakdown

### 1️⃣ Setup & Catalog (1_setup_catalog/)

#### `utilities.py`
**Purpose:** Centralized schema definitions and shared utilities  
**Key Definitions:**
```python
bronze_schema = "bronze"
silver_schema = "silver"
gold_schema = "gold"
```
- Provides schema naming conventions for all ETL jobs
- Used across all dimension and fact processing modules

#### `dim_date_table_creation.py`
**Purpose:** Create a dimensional date table for time-based analytics  
**Key Functionality:**
- Generates monthly grain dates from 2024-01-01 to 2025-12-01
- Creates derived columns:
  - `date_key` (YYYYMM format, e.g., 202401)
  - `year`, `month_name`, `month_short_name`
  - `quarter`, `year_quarter`
- Saves to `fmcg.gold.dim_date` Delta table
- Enables efficient joins for time-based reporting

**Output Schema:**
```
month_start_date: date
date_key: integer (yyyyMM)
year: integer
month_name: string
month_short_name: string
quarter: string (e.g., Q1)
year_quarter: string (e.g., 2024-Q1)
```

---

### 2️⃣ Dimension Data Processing (2_dimension_data_processing/)

#### `1_customer_data_processing.py`
**Purpose:** Process and standardize customer dimension data  
**Data Source:** Azure ADLS Gen2 - `customers/*.csv`

**Processing Steps:**

| Layer | Transformations |
|-------|-----------------|
| **Bronze** | Load CSV with metadata (file_name, file_size, read_timestamp) |
| **Silver** | • Remove duplicates (by customer_id)<br>• Trim whitespace from names<br>• Standardize city names (fix misspellings)<br>• Handle null cities with business-confirmed mappings<br>• Apply proper casing to customer names<br>• Create composite "customer" field |
| **Gold** | Select essential columns, stage for merge<br>Merge with parent company dim_customers table |

**Key Transformations:**
- **City Mapping:** Bengaluruu → Bengaluru, Hyderabadd → Hyderabad, etc.
- **Composite Key:** Creates "CustomerName-City" field
- **Static Attributes:** Market (India), Platform (Sports Bar), Channel (Acquisition)

**Output Tables:**
- `fmcg.bronze.customers`
- `fmcg.silver.customers`
- `fmcg.gold.sb_dim_customers` (SportsBar customer dimension)
- Merges into `fmcg.gold.dim_customers` (parent company table)

---

#### `2_products_data_processing.py`
**Purpose:** Process and standardize product dimension data  
**Data Source:** Azure ADLS Gen2 - `products/*.csv`

**Processing:** Similar structure to customer processing
- Bronze → Raw data ingestion
- Silver → Cleaning, standardization, deduplication
- Gold → Business-ready product dimension

---

#### `3_pricing_data_processing.py`
**Purpose:** Process pricing dimension data  
**Data Source:** Azure ADLS Gen2 - `pricing/*.csv`

**Processing:** Standard medallion architecture pipeline for pricing information

---

### 3️⃣ Fact Data Processing (3_fact_data_processing/)

#### `1_full_load_fact.py`
**Purpose:** Initial full load of order/transaction facts

**Data Source:** 
- Landing path: `orders/landing/*.csv`
- Processed path: `orders/processed/` (post-ingestion)

**Bronze Layer:**
- Read CSV files from landing directory
- Add metadata columns (file_name, file_size, read_timestamp)
- Append to `fmcg.bronze.orders` with Change Data Feed enabled

**Silver Layer:**
- **Filter:** Keep only rows with non-null `order_qty`
- **Customer ID Cleanup:** Keep numeric IDs, set invalid to "999999"
- **Date Parsing:** Remove weekday names, parse dates in multiple formats
  - Supports: yyyy/MM/dd, dd-MM-yyyy, dd/MM/yyyy, MMMM dd, yyyy
- **Deduplication:** Drop duplicates by (order_id, order_placement_date, customer_id, product_id, order_qty)
- **Type Casting:** Convert product_id to string
- **Join with Products:** Inner join with `fmcg.silver.products` table
- **Merge or Insert:** Uses MERGE logic for upserts into `fmcg.silver.orders`

**Gold Layer:**
- **Aggregation:** Transform daily orders to monthly grain
  - Group by: month_start, product_code, customer_code
  - Aggregate: SUM(sold_quantity)
- **Merge with Parent:** Update parent company's `fmcg.gold.fact_orders` table
- **Output:** `fmcg.gold.sb_fact_orders` (SportsBar orders fact table)

**File Movement:**
- After successful processing, moves files from `orders/landing/` to `orders/processed/`

---

#### `2_incremental_load_fact.py`
**Purpose:** Daily incremental load of new/updated order facts

**Processing:**
- Follows same transformation logic as full load
- Uses MERGE operations for efficient incremental updates
- Supports correction of previous months or status changes
- Handles daily data appends

---

## 🔄 Data Pipeline Flow

### End-to-End Order Processing Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                             │
│  (Spreadsheets, Cloud Drives, APIs, CSV Exports)               │
└──────────────────────┬──────────────────────────────────────────┘
                       │
                       ▼
        ┌──────────────────────────┐
        │    AZURE ADLS Gen2       │
        │  Landing Directories    │
        │ (orders, customers, etc) │
        └──────────────┬───────────┘
                       │
        ┌──────────────┴──────────────┐
        │                             │
        ▼                             ▼
    ┌─────────────┐            ┌──────────────┐
    │   ORDERS    │            │  DIMENSIONS  │
    │   PIPELINE  │            │   PIPELINE   │
    └──────┬──────┘            └──────┬───────┘
           │                          │
    ┌──────▼──────────┐       ┌───────▼─────────┐
    │     BRONZE      │       │     BRONZE      │
    │  fmcg.bronze.   │       │  fmcg.bronze.   │
    │ orders (raw)    │       │ customers/...   │
    └──────┬──────────┘       └───────┬─────────┘
           │                          │
    ┌──────▼──────────┐       ┌───────▼─────────┐
    │     SILVER      │       │     SILVER      │
    │  fmcg.silver.   │       │  fmcg.silver.   │
    │ orders (clean)  │       │ customers/...   │
    │                 │       │  (clean & std)  │
    │  Joins with     │       │                 │
    │  products       │       │ Merges with     │
    └──────┬──────────┘       │ parent tables   │
           │                  └─────────────────┘
    ┌──────▼──────────────────┐
    │        GOLD             │
    │  (Analytics Ready)      │
    │                         │
    │ fmcg.gold.sb_fact_      │
    │ orders (monthly grain)  │
    │                         │
    │ Merges with parent      │
    │ fmcg.gold.fact_orders   │
    └─────────────────────────┘
           │
           ▼
    ┌──────────────┐
    │  REPORTING & │
    │  ANALYTICS   │
    │  DASHBOARDS  │
    └──────────────┘
```

### Data Quality Steps at Each Layer

**Bronze:**
- Raw data capture with metadata
- File tracking for auditability

**Silver:**
- Deduplication
- Type standardization
- Null value handling
- Format standardization (dates, text)
- Business rule application
- Join with reference tables

**Gold:**
- Business-level aggregation
- Grain transformation (daily → monthly)
- Merge with parent company data
- Analytics-ready structure

---

## 🔑 Key Concepts

### Medallion Architecture

```
BRONZE (Raw)
├─ Purpose: Data ingestion & lineage
├─ Data State: Untransformed, original structure
├─ Update Pattern: Append daily
└─ User: Data engineers, auditors

    ▼

SILVER (Cleaned)
├─ Purpose: Single source of truth
├─ Data State: Deduplicated, standardized
├─ Update Pattern: Merge (upsert)
└─ User: Analytics engineers, analysts

    ▼

GOLD (Business Ready)
├─ Purpose: Analytics & reporting
├─ Data State: Aggregated, business logic applied
├─ Update Pattern: Merge (upsert)
└─ User: Analysts, BI tools, dashboards
```

### Delta Lake & MERGE Operations
- **ACID Compliance:** Ensures data consistency
- **Schema Enforcement:** Prevents schema drift
- **Change Data Feed:** Tracks changes for auditing
- **Merge Strategy:** Efficiently handles upserts
  - `whenMatchedUpdateAll()` - Update existing records
  - `whenNotMatchedInsertAll()` - Insert new records

### Change Data Feed (CDF)
All tables are created with `delta.enableChangeDataFeed = true`:
```python
.option("delta.enableChangeDataFeed", "true")
```
- Enables tracking of row-level changes
- Useful for change capture and audit trails

---

## 🛠️ Implementation Details

### Storage Paths

**ADLS Gen2 Structure:**
```
abfss://container-de-practice@adlsgen2narayan.dfs.core.windows.net/
├── orders/
│   ├── landing/        (incoming CSV files)
│   └── processed/      (completed files)
├── customers/
│   └── *.csv
├── products/
│   └── *.csv
└── pricing/
    └── *.csv
```

### Database Catalog Structure

```
fmcg (Catalog)
├── bronze (Schema)
│   ├── customers
│   ├── products
│   ├── pricing
│   └── orders
├── silver (Schema)
│   ├── customers
│   ├── products
│   ├── pricing
│   └── orders
└── gold (Schema)
    ├── dim_date
    ├── dim_customers → sb_dim_customers
    ├── dim_products → sb_dim_products
    ├── dim_pricing → sb_dim_pricing
    ├── sb_fact_orders
    └── fact_orders (merged parent table)
```

### Parameters & Configuration

**Databricks Widgets (UI Parameters):**
```python
catalog = "fmcg"              # Target catalog
data_source = "customers"     # Data source name (customers, products, orders, etc.)
```

**Key Configurations:**
- Date Range: 2024-01-01 to 2025-12-01 (for date dimension)
- Allowed Cities: ['Bengaluru', 'Hyderabad', 'New Delhi']
- Invalid Customer ID Default: 999999
- Aggregation Grain (Orders): Monthly
- File Format: Delta, Delta Merge enabled

### Data Type Conversions

| Field | Bronze Type | Silver Type | Purpose |
|-------|-------------|------------|---------|
| customer_id | Original | String | Standardization |
| product_id | Original | String | Consistency |
| order_qty | Original | Numeric | Aggregation |
| order_placement_date | String (multiple formats) | Date | Reporting |
| city | String | String (standardized) | Dimension quality |

---

## 📈 Known Issues & Fixes

### Documented Fixes

1. **Customer City Mapping:** Hardcoded fixes for specific customers (IDs: 789403, 789420, 789521, 789603, 789421, 789422, 789522)
   - Status: Business team confirmed
   
2. **Date Parsing:** Multiple format support to handle inconsistent date formats
   - Supports: `yyyy/MM/dd`, `dd-MM-yyyy`, `dd/MM/yyyy`, `MMMM dd, yyyy`
   - Also removes weekday prefix if present

3. **Data Quality:** Dimension reduction based on allowed values
   - City validation against allowed list
   - Customer name standardization

---

## 🚀 Deployment & Orchestration

### Workflow Execution
- **Platform:** Databricks Workflows
- **Trigger:** Daily schedule (to be configured)
- **Execution Order:** 
  1. Date dimension creation (if needed)
  2. Dimension processing (customers, products, pricing)
  3. Fact processing (orders full/incremental load)

### Integration with Azure Data Factory
- Configuration file: `factory/nsahu-de-adf.json`
- Supports orchestration of Databricks notebook executions
- Integration Runtime: AutoResolveIntegrationRuntime

---

## 📝 File Movement & Audit Trail

### Order Processing Lifecycle
```
Landing Directory (raw CSV)
    │
    ├─ Read → Bronze → Silver → Gold
    │
Processed Directory (archive after success)
```

**Metadata Tracked:**
- `read_timestamp` - When data was ingested
- `file_name` - Source file name
- `file_size` - File size in bytes
- Change Data Feed - Row-level changes

---

## 🔍 Data Quality Metrics

### Deduplication
- **Customer Processing:** Remove duplicates by `customer_id`
- **Order Processing:** Remove duplicates by `(order_id, order_placement_date, customer_id, product_id, order_qty)`

### Validation Rules
- Customer ID: Must be numeric (or default to 999999)
- Order Qty: Must be non-null
- City: Must be in allowed list or null
- Order Date: Must be parseable to date format

---

## 📚 Additional Configuration Files

### Azure Data Factory
- **File:** `factory/nsahu-de-adf.json`
- **Purpose:** Orchestration configuration for Databricks pipeline

### Integration Runtime
- **File:** `integrationRuntime/AutoResolveIntegrationRuntime.json`
- **Purpose:** Compute environment for data integration

### Managed Virtual Network
- **File:** `managedVirtualNetwork/default.json`
- **Purpose:** Network isolation for data factory

### Spark Setup
- **Files:** `Spark/PreReqInstallation/`
  - `InstallationSteps.txt` - Installation guide
  - `SparkOperation.txt` - Operational guidelines

---

## 📞 Next Steps & Recommendations

### Phase 1: Validation
- [ ] Validate data quality in Silver layer
- [ ] Confirm dimension counts match source
- [ ] Verify foreign key relationships in Gold

### Phase 2: Production Readiness
- [ ] Set up automated scheduling in Databricks Workflows
- [ ] Configure alerting for pipeline failures
- [ ] Implement monitoring dashboards
- [ ] Document SLAs for data freshness

### Phase 3: Enhancement
- [ ] Add incremental load optimization
- [ ] Implement data lineage tracking
- [ ] Set up cost optimization monitoring
- [ ] Create user training documentation

---

## 📎 References

- **Project Plan:** `Databricks_project_1/README.MD`
- **Root Documentation:** `README.md`
- **Platform:** Databricks Free Edition / Databricks Community
- **Storage:** Azure Data Lake Storage Gen2 (ADLS Gen2)
- **Delta Lake:** [https://delta.io/](https://delta.io/)

---

**Document Version:** 1.0  
**Last Updated:** December 2, 2025  
**Author:** Documentation Auto-generated
