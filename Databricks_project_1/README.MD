# 🏢 SportsBar Order Data Pipeline - Databricks Lakehouse Project

> **A Production-Grade Data Engineering Portfolio Project**  
> Building a scalable, ACID-compliant data warehouse for FMCG analytics using the Medallion Architecture

---

## 📋 Quick Links
- [Project Goal](#1-project-goal-and-context-)
- [Architecture](#2-architecture-and-technology-stack-)
- [Pipeline Design](#3-pipeline-design-bronze-silver-gold-layers)
- [Implementation](#4-implementation-strategy-and-next-steps-)
- [How to Use](#-how-to-use-this-project)
- [Project Structure](#-project-structure)

---

## 1. Project Goal and Context 🎯

### 🚀 Objective
Implement a **unified, scalable data pipeline** for the recently merged SportsBar entity, leveraging the **Databricks Lakehouse Platform**. Integrate SportsBar's order details from diverse, unstructured sources into a structured, reliable data warehouse for the FMCG (Fast-Moving Consumer Goods) analytics team.

### 📊 The Challenge
- **Data scattered across:** Spreadsheets, cloud drives, WhatsApp exports, inconsistent APIs
- **No single source of truth** for order tracking
- **Manual processes** for data consolidation
- **Scalability concerns** with growing data volumes

### ✅ Solution
Create a **production-grade, automated data pipeline** that:
- ✨ Centralizes all order data in one reliable source
- 🔄 Automates daily incremental data loads
- 🛡️ Ensures data quality and consistency
- 📈 Enables real-time analytics and forecasting
- 🔐 Maintains audit trails and data lineage

---

## 2. Architecture and Technology Stack 🏗️

### Core Components

| **Component** | **Technology** | **Purpose** |
|:---|:---|:---|
| **Cloud Platform** | ☁️ **Microsoft Azure** | Enterprise-grade cloud infrastructure |
| **Data Platform** | 📊 **Databricks Lakehouse** | Unified analytics platform with compute & storage |
| **Storage Layer** | 🗄️ **Delta Lake** | ACID-compliant, versioned data lake format |
| **Compute Engine** | ⚙️ **Apache Spark** | Distributed parallel processing |
| **Processing Language** | 🐍 **Python + SQL** | Data transformation and orchestration |
| **Storage Account** | 📦 **Azure Data Lake Gen2 (ADLS)** | Scalable cloud data storage |
| **Orchestration** | ⏰ **Databricks Workflows** | Automated pipeline scheduling |
| **Integration** | 🔗 **Azure Data Factory** | Enterprise data orchestration |

### Key Technologies Deep Dive

**Delta Lake Benefits:**
- ✅ ACID Transactions - Consistency across reads/writes
- ✅ Schema Enforcement - Prevents data corruption
- ✅ Change Data Feed (CDF) - Audit and change tracking
- ✅ Merge Operations - Efficient upserts for incremental loads
- ✅ Time Travel - Point-in-time data recovery

**Medallion Architecture:**
- **Bronze Layer** → Raw data ingestion with full lineage
- **Silver Layer** → Cleaned, deduplicated, conformed data
- **Gold Layer** → Business-ready, aggregated analytics data




## 3. Pipeline Design: Bronze, Silver, Gold Layers 🔄

### 3.1. Bronze Layer (Raw Ingestion)
**Purpose:** Capture raw data with full lineage for auditability  
**Source:** Direct ingestion from SportsBar's original locations (spreadsheets, cloud drives, APIs, etc.)  
**Data Structure:** Raw, untransformed data  
**Load Type:** Daily Incremental Load  

**Key Characteristics:**
- Minimal transformation, captures original state
- Includes metadata: `read_timestamp`, `file_name`, `file_size`
- Enables data recovery and audit trails
- Change Data Feed (CDF) enabled for tracking

**Implementation:**
```python
df.write.format("delta")
    .option("delta.enableChangeDataFeed", "true")
    .mode("append")
    .saveAsTable(bronze_table)
```

---

### 3.2. Silver Layer (Cleaned & Conformed)
**Purpose:** Provide a single, clean, and reliable source of truth  
**Transformation:** Apply cleaning, standardization, and conformity rules  

**Data Quality Steps:**
- ✓ Deduplication by business keys
- ✓ Type standardization and casting
- ✓ Null value handling with business rules
- ✓ Date format standardization (supports multiple formats)
- ✓ Text cleanup (trim, case conversion)
- ✓ Reference data joins (e.g., products, customers)
- ✓ Dimension reduction based on allowed values

**Load Type:** Daily Incremental Merge  
**Key Function:** Uses Delta Lake's MERGE functionality for efficient upserts

**Implementation:**
```python
delta_table.alias("target").merge(
    source=df_new.alias("source"),
    condition="target.key = source.key"
).whenMatchedUpdateAll()
 .whenNotMatchedInsertAll()
 .execute()
```

**Validation Examples:**
- Customer ID: Numeric validation (or default to 999999)
- City: Must be in allowed list (Bengaluru, Hyderabad, New Delhi)
- Order Qty: Must be non-null
- Dates: Parse multiple formats with fallback

---

### 3.3. Gold Layer (Analytics Ready)
**Purpose:** Power reporting, dashboards, and forecasting  
**Transformation:** Business logic and aggregations tailored for analytics

**Business Logic:**
- ✓ Joining with reference tables
- ✓ Creating aggregated facts (daily sales, product performance)
- ✓ Grain transformation (daily → monthly for orders)
- ✓ Business key mapping and composite dimensions

**Data Structure:** Highly structured, optimized for read performance  
**Key Function:** Powers analytics and decision-making

**Output Example:**
```
Monthly aggregated orders:
- date (month_start)
- product_code
- customer_code
- sold_quantity (SUM)
```

**Merge with Parent Company:**
```python
gold_parent_delta.alias("parent").merge(
    df_monthly.alias("child"),
    condition="parent.date = child.date AND parent.product_code = child.product_code"
).whenMatchedUpdateAll()
 .whenNotMatchedInsertAll()
 .execute()
```

---

## 4. Implementation Strategy and Next Steps 🚀

### Phase 1: Planning and Validation ✓
- [x] Detailed analysis of data sources
- [x] Architecture validation with Medallion pattern
- [x] Phased migration planning starting with orders

### Phase 2: Execution 🔄
- [x] Ingestion Framework for messy source data
- [x] Delta Lake MERGE logic implementation
- [x] Data quality rules implementation
- [ ] Automated scheduling setup
- [ ] Monitoring and alerting

### Phase 3: Adoption and Handover 📋
- [ ] Training plan for data engineering team
- [ ] Documentation for stakeholders
- [ ] Performance tuning and optimization
- [ ] Cost monitoring

---

## 📁 Project Structure & Modules

```
Databricks_project_1/
│
├── 📄 README.MD (this file)
│
├── 1️⃣ 1_setup_catalog/          # ⚙️ SETUP & CONFIGURATION LAYER
│   ├── utilities.py                    # Shared schema definitions
│   ├── setup.py                        # Initial catalog setup
│   └── dim_date_table_creation.py      # Date dimension creation
│
├── 2️⃣ 2_dimension_data_processing/   # 📊 DIMENSION TABLES (Bronze → Silver → Gold)
│   ├── 1_customer_data_processing.py   # Customer dimension with deduplication & standardization
│   ├── 2_products_data_processing.py   # Product dimension ETL
│   └── 3_pricing_data_processing.py    # Pricing dimension ETL
│
└── 3️⃣ 3_fact_data_processing/        # 📈 FACT TABLES (Orders - Daily Load)
    ├── 1_full_load_fact.py             # Initial full load + monthly aggregation
    └── 2_incremental_load_fact.py      # Daily incremental loads
```

### Module Responsibilities

| Module | Purpose | Input | Output |
|--------|---------|-------|--------|
| **Setup** | Initialize catalog, schemas, utilities | - | Schemas, shared functions |
| **Date Dimension** | Create time-based analytics table | Date range config | Monthly grain date table |
| **Customer Processing** | Integrate & standardize customer data | Raw customer CSV | Deduplicated, standardized customers |
| **Product Processing** | Process product catalog | Raw product CSV | Standardized product dimensions |
| **Pricing Processing** | Handle pricing information | Raw pricing CSV | Standardized pricing tables |
| **Order Full Load** | Initial order data ingestion | Raw orders CSV | Bronze → Silver → Gold |
| **Order Incremental Load** | Daily order updates | New orders CSV | Daily incremental updates |

---

## 🔄 Data Flow Diagram

```
┌─────────────────────────────┐
│    AZURE ADLS Gen2          │
│   (Raw Data Sources)        │
│ • orders/landing/*.csv      │
│ • customers/*.csv           │
│ • products/*.csv            │
│ • pricing/*.csv             │
└────────────┬────────────────┘
             │
             ▼
    ┌────────────────┐
    │   BRONZE       │ ← Raw data ingestion + metadata
    │   (Raw)        │
    └────────┬───────┘
             │
             ▼
    ┌────────────────┐
    │   SILVER       │ ← Cleaned, deduplicated, standardized
    │   (Conformed)  │
    └────────┬───────┘
             │
             ▼
    ┌────────────────┐
    │   GOLD         │ ← Business logic, aggregation
    │   (Analytics)  │
    └────────┬───────┘
             │
             ▼
┌─────────────────────────────┐
│  REPORTING & DASHBOARDS     │
│ (BI Tools, Analysts)        │
└─────────────────────────────┘
```

---

## 🛠️ How to Use This Project

### Prerequisites
- Databricks Workspace access
- Azure Data Lake Storage Gen2 (ADLS Gen2) configured
- Python 3.x runtime
- Delta Lake enabled on Databricks cluster

### Setup Steps

1. **Initialize Catalog**
   ```
   Run: 1_setup_catalog/utilities.py
   Purpose: Define schemas (bronze, silver, gold)
   ```

2. **Create Date Dimension**
   ```
   Run: 1_setup_catalog/dim_date_table_creation.py
   Output: fmcg.gold.dim_date table
   ```

3. **Process Dimensions** (in order)
   ```
   Run: 2_dimension_data_processing/1_customer_data_processing.py
   Run: 2_dimension_data_processing/2_products_data_processing.py
   Run: 2_dimension_data_processing/3_pricing_data_processing.py
   ```

4. **Load Fact Data**
   ```
   First run (initial load):
   Run: 3_fact_data_processing/1_full_load_fact.py
   
   Subsequent runs (daily incremental):
   Run: 3_fact_data_processing/2_incremental_load_fact.py
   ```

### Configuration Parameters

Each notebook accepts these Databricks widgets:

```python
catalog = "fmcg"                    # Target catalog name
data_source = "customers"           # Data source name (customers, products, orders, etc.)
```

### Expected Data Sources

Place your data files in these ADLS locations:
```
abfss://container-de-practice@adlsgen2narayan.dfs.core.windows.net/
├── customers/
│   └── *.csv (columns: customer_id, customer_name, city)
├── products/
│   └── *.csv (columns: product_id, product_code, product_name, ...)
├── pricing/
│   └── *.csv (columns: product_id, price, ...)
└── orders/
    └── landing/ (incoming CSV files)
        └── *.csv (columns: order_id, customer_id, product_id, order_qty, order_placement_date)
```

---

## 📊 Key Features & Highlights

### 🎯 Data Quality Framework
- **Deduplication:** Remove duplicate records by business keys
- **Type Standardization:** Enforce consistent data types
- **Validation:** Multiple format support for dates, numeric IDs
- **Null Handling:** Business rule-based null value management
- **Reference Data Joins:** Ensure referential integrity

### 🔄 Incremental Load Strategy
- **Change Data Feed (CDF):** Enabled on all tables for audit trails
- **Delta Merge Operations:** Efficient upserts with `whenMatchedUpdateAll()` and `whenNotMatchedInsertAll()`
- **File Tracking:** Move processed files to archive directory
- **Daily Scheduling:** Supports automated daily incremental loads

### 📈 Aggregation Logic
- **Grain Transformation:** Daily orders → Monthly aggregates
- **Cross-Company Merge:** Integration with parent company data models
- **Composite Keys:** Multi-column join keys for accuracy

### 🛡️ Data Lineage & Auditability
- **Metadata Tracking:** `read_timestamp`, `file_name`, `file_size`
- **Change Tracking:** Delta Lake CDF for row-level changes
- **Time Travel:** Point-in-time data recovery capability

---

## 💡 Technical Highlights

### 1. Multi-Format Date Parsing
Handles multiple date formats automatically:
- `yyyy/MM/dd` (2025-01-15)
- `dd-MM-yyyy` (15-01-2025)
- `dd/MM/yyyy` (15/01/2025)
- `MMMM dd, yyyy` (January 15, 2025)
- With optional weekday prefix removal

### 2. Business Rule Application
Example: City standardization with hardcoded business corrections
```python
city_mapping = {
    'Bengaluruu': 'Bengaluru',
    'Hyderabadd': 'Hyderabad',
    ...
}
allowed_cities = ['Bengaluru', 'Hyderabad', 'New Delhi']
```

### 3. Monthly Grain Transformation
Converts daily order details to monthly aggregates:
```python
.groupBy("month_start", "product_code", "customer_code")
.agg(F.sum("sold_quantity").alias("sold_quantity"))
```

---

## 📚 Documentation Structure

Each notebook includes:
- **Purpose Statement:** What the notebook does
- **Data Sources:** Input data location and format
- **Transformation Steps:** Detailed comments for each step
- **Output Tables:** Target tables created/updated
- **Quality Checks:** Validation results and metrics

---

## 🚀 Performance Considerations

- **Partition Strategy:** Consider partitioning Bronze tables by date
- **Clustering:** Use composite keys for efficient joins
- **Caching:** Cache frequently accessed dimension tables
- **Cost Optimization:** Monitor cluster utilization and downscale off-peak

---

## 📋 Known Issues & Mitigations

### Issue 1: Inconsistent Date Formats
**Solution:** Multi-format parsing with `try_to_date()` and `coalesce()`

### Issue 2: Null Cities in Customer Data
**Solution:** Hardcoded business mappings validated with stakeholders

### Issue 3: Invalid Customer IDs
**Solution:** Default to 999999 for non-numeric IDs, maintain referential integrity

---

## 🔐 Security & Best Practices

- ✅ Use Databricks Secrets for ADLS credentials
- ✅ Implement schema-level permissions
- ✅ Enable Change Data Feed for compliance
- ✅ Archive processed files for audit trails
- ✅ Use service principals for automated runs

---

## 📈 Future Enhancements

- [ ] Real-time streaming with Kafka/Event Hub
- [ ] ML-based data quality scoring
- [ ] Advanced monitoring with Databricks SQL Alerts
- [ ] Automated documentation generation
- [ ] Cost allocation and chargeback models
- [ ] Data governance with Unity Catalog
- [ ] Advanced lineage tracking

---

## 🤝 Contributing

To extend this project:

1. **Add new data sources:** Follow the medallion pattern
2. **Add new dimensions:** Create new processing notebooks
3. **Optimize performance:** Profile slow operations
4. **Document changes:** Update README and inline comments

---

## 📞 Support & Questions

For questions about the pipeline:
- Review `WORKSPACE_DOCUMENTATION.md` for detailed module breakdown
- Check inline notebook comments for transformation logic
- Refer to Databricks documentation for framework-specific issues

---

## 📄 License & Attribution

**Project Type:** Educational / Portfolio  
**Last Updated:** December 2, 2025  
**Author:** Narayan Sahu  
**Repository:** DataEnginering  

---

## 🎓 Learning Outcomes

Working through this project, you'll learn:

✅ **Databricks Ecosystem:** Notebooks, Workflows, SQL analytics  
✅ **Delta Lake:** ACID compliance, merge operations, change data feed  
✅ **Apache Spark:** DataFrame transformations, distributed processing  
✅ **Data Architecture:** Medallion pattern, dimension modeling  
✅ **Data Quality:** Validation, deduplication, standardization  
✅ **Cloud Integration:** Azure ADLS Gen2, service principals  
✅ **Production Practices:** Error handling, auditing, monitoring  

---

**🌟 Start with:** `1_setup_catalog/utilities.py` → `dim_date_table_creation.py` → Dimensions → Facts