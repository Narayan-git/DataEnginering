# 🚀 Quick Reference Guide - Data Engineering Portfolio

## 📍 Start Here - 5 Minute Overview

### What This Project Is
A production-grade data pipeline that transforms messy business order data into clean, aggregated analytics data using:
- **Platform:** Databricks (Spark + Delta Lake)
- **Architecture:** Medallion (Bronze → Silver → Gold)
- **Cloud:** Microsoft Azure

### The Pipeline Journey
```
Raw CSV Files (Landing) 
  ↓ (Bronze Layer)
Raw Data Table (full lineage)
  ↓ (Silver Layer)
Clean Data Table (deduped, validated, joined)
  ↓ (Gold Layer)
Monthly Aggregates (ready for BI tools)
  ↓ (Merge to Parent)
Company-wide Facts Table
```

---

## 📚 Essential Reading Order

| Time | Document | Why Read |
|------|----------|----------|
| 5 min | README.MD | Understand architecture |
| 10 min | This guide | Quick reference |
| 15 min | PORTFOLIO_GUIDE.md intro | Interview prep |
| 30 min | WORKSPACE_DOCUMENTATION.md | Technical details |
| 2 hours | Enhanced notebooks | Deep learning |

---

## 🎯 Key Code Patterns to Know

### 1. Multi-Format Date Parsing (🌟 Interview Highlight)
```python
# Handles 4 different date formats automatically
.withColumn(
    "date",
    F.coalesce(
        F.try_to_date("date", "yyyy/MM/dd"),
        F.try_to_date("date", "dd-MM-yyyy"),
        F.try_to_date("date", "dd/MM/yyyy"),
        F.try_to_date("date", "MMMM dd, yyyy"),
    )
)
```
**Why:** Real data is messy. Shows defensive programming.

### 2. Delta MERGE for Efficient Updates
```python
# Upsert pattern (update if exists, insert if new)
delta_table.alias("target").merge(
    df_new.alias("source"),
    "target.id = source.id"
).whenMatchedUpdateAll()
 .whenNotMatchedInsertAll()
 .execute()
```
**Why:** ACID compliance, efficient incremental loads.

### 3. Data Quality Rules
```python
# Validate customer ID (keep numeric, default invalid to 999999)
.withColumn(
    "customer_id",
    F.when(F.col("customer_id").rlike("^[0-9]+$"), F.col("customer_id"))
     .otherwise("999999")
     .cast("string")
)
```
**Why:** Handles data quality issues gracefully.

### 4. Grain Transformation (Daily → Monthly)
```python
# Aggregate daily orders to monthly grain
df.groupBy("month_start", "product_code", "customer_code")\
  .agg(F.sum("sold_quantity").alias("sold_quantity"))
```
**Why:** Typical analytics requirement, good for performance.

### 5. File Archival Pattern
```python
# Move processed files to prevent reprocessing
files = dbutils.fs.ls(landing_path)
for file_info in files:
    dbutils.fs.mv(
        file_info.path,
        f"{processed_path}/{file_info.name}",
        True
    )
```
**Why:** Production practice, maintains audit trail.

---

## 💼 Interview Elevator Pitch (30 seconds)

> "I built a production-grade data pipeline using Databricks that processes daily orders for a retail company. It implements the Medallion Architecture with Bronze (raw), Silver (clean), and Gold (analytics) layers using Delta Lake for ACID compliance. The pipeline handles multiple data formats, applies business rules for data quality, and efficiently merges incremental loads using Delta MERGE operations. It aggregates daily orders to monthly grain for reporting and integrates with the parent company's master data."

---

## ✅ Quick Interview Checklist

### Can You Explain:
- ✅ What is Medallion Architecture?
  - **Answer:** Bronze (raw) → Silver (clean) → Gold (analytics)
  - **Located:** README.MD, all notebooks

- ✅ How does Delta Lake help?
  - **Answer:** ACID transactions, MERGE operations, Change Data Feed
  - **Located:** dim_date_table_creation.ipynb, full_load_fact.ipynb

- ✅ How do you handle bad data?
  - **Answer:** Validation, deduplication, standardization with business rules
  - **Located:** customer_data_processing.ipynb (10-step process)

- ✅ Why MERGE instead of INSERT?
  - **Answer:** Efficient for both inserts and updates, ACID compliant
  - **Located:** Full and incremental load notebooks

- ✅ How does this scale?
  - **Answer:** Spark distributed processing, partitioning, incremental loads
  - **Located:** PORTFOLIO_GUIDE.md optimization section

### Code You Can Explain:
- ✅ Multi-format date parsing (line ~line with try_to_date)
- ✅ Deduplication logic (dropDuplicates with business keys)
- ✅ MERGE operation syntax (whenMatched / whenNotMatched)
- ✅ Business rules (customer city mapping, ID validation)
- ✅ Aggregation (groupBy and sum for monthly grain)

---

## 🎓 Learning Progression (3+ hours)

### Phase 1: Setup & Configuration (15 min)
**Read:** `utilities.ipynb`, `dim_date_table_creation.ipynb`
**Learn:** How to organize code, create dimensions

### Phase 2: Dimension Processing (45 min)
**Read:** `customer_data_processing.ipynb`
**Learn:** Data quality frameworks, 10-step transformation process

### Phase 3: Fact Processing (60 min)
**Read:** `full_load_fact.ipynb`
**Learn:** Advanced transformations, multi-format parsing, aggregations

### Phase 4: Integration & Patterns (30 min)
**Read:** All notebooks with integration focus
**Learn:** Cross-notebook patterns, parameterization, reusability

### Phase 5: Presentation (30 min)
**Read:** PORTFOLIO_GUIDE.md
**Practice:** Your answers to 8 interview questions

---

## 📊 Documentation Map

```
Project Root/
├── README.MD                    ← Start here (architecture)
├── PORTFOLIO_GUIDE.md          ← Interview prep (must read!)
├── WORKSPACE_DOCUMENTATION.md  ← Technical deep dive
├── ENHANCEMENT_SUMMARY.md      ← What was improved
│
├── 1_setup_catalog/
│   ├── utilities.ipynb         ← 200 lines docs
│   └── dim_date_table.ipynb    ← 400 lines docs
│
├── 2_dimension_data_processing/
│   └── 1_customer...ipynb      ← 800 lines docs (most detailed)
│
└── 3_fact_data_processing/
    ├── 1_full_load...ipynb     ← 600 lines docs
    └── 2_incremental...ipynb   ← Ready for enhancement
```

**Total:** 4,000+ lines of professional documentation

---

## 🌟 Standout Talking Points

### Technical Depth
- "I handle 4 different date formats with fallback logic"
- "I use Delta MERGE for efficient incremental loads"
- "I implement a 10-step data quality process"

### Business Understanding
- "I work with business team to confirm data corrections"
- "I map customer cities to validated lists"
- "I aggregate daily orders to monthly grain per requirements"

### Production Practices
- "I archive processed files to prevent reprocessing"
- "I track metadata for audit trails"
- "I validate data at each layer before proceeding"

### Problem Solving
- "Here's how I handle inconsistent data formats..."
- "My approach to deduplication balances accuracy and performance..."
- "I validate before processing to catch issues early..."

---

## ⚡ Quick Facts About the Project

| Aspect | Detail |
|--------|--------|
| **Platform** | Databricks Lakehouse |
| **Storage** | Azure Data Lake Gen2 |
| **Format** | Delta Lake |
| **Language** | PySpark (Python + SQL) |
| **Architecture** | Medallion (3-layer) |
| **Key Pattern** | MERGE for incremental loads |
| **Data Quality Steps** | 10+ validation rules |
| **Date Formats Supported** | 4 different formats |
| **Aggregation Grain** | Daily → Monthly |
| **Deduplication Keys** | Business-based (order_id, customer, product) |

---

## 🎯 Common Interview Questions & Your Answers

### Q1: "Walk me through your pipeline"
**Your Answer:**
"Raw orders come from CSV files in Azure ADLS. The Bronze layer loads them as-is with metadata. The Silver layer applies 10 data quality steps including date parsing (handles 4 formats), customer ID validation, deduplication, and joins with product data. The Gold layer aggregates to monthly grain per business requirements. Finally, I merge SportsBar data into the parent company's master fact table using Delta MERGE."

**Location:** README.MD, WORKSPACE_DOCUMENTATION.md

### Q2: "How do you handle data quality?"
**Your Answer:**
"I have a systematic 10-step process: null validation, type standardization, format parsing, deduplication, business rule application, joins with reference data, and dimensional validation. I show problems to the business team (like missing city data) and apply approved corrections. All changes are tracked with metadata."

**Location:** customer_data_processing.ipynb (shows all 10 steps)

### Q3: "Explain your use of Delta MERGE"
**Your Answer:**
"Delta MERGE is perfect for incremental loads. It checks if a record exists (by key), updates it if it does (whenMatchedUpdateAll), or inserts if it doesn't (whenNotMatchedInsertAll). This is efficient, ACID-compliant, and handles both daily incremental loads and cross-company master data merges."

**Location:** full_load_fact.ipynb, customer_data_processing.ipynb

### Q4: "What's tricky about the date parsing?"
**Your Answer:**
"Orders come from different systems with different date formats. Some have weekday prefixes that need stripping first. I use try_to_date with multiple formats in a coalesce, so it tries each format and returns the first non-null result. If none match, it stays null for investigation."

**Location:** full_load_fact.ipynb (shows exact code)

### Q5: "How do you scale this?"
**Your Answer:**
"I use Spark for distributed processing. The incremental load pattern with MERGE is efficient for daily runs. For larger volumes, I'd partition by date, use clustering on join keys, and cache dimension tables. The architecture naturally supports scaling from thousands to billions of rows."

**Location:** PORTFOLIO_GUIDE.md optimization section

---

## 🚀 Before Your Interview

### This Week
- [ ] Read README.MD (5 min)
- [ ] Read PORTFOLIO_GUIDE.md (20 min)
- [ ] Run through this Quick Reference (5 min)
- [ ] Review 5 Key Code Patterns (10 min)
- [ ] Practice elevator pitch in mirror (5 min)

### Day Before
- [ ] Review PORTFOLIO_GUIDE.md again
- [ ] Practice 5 interview Q&A
- [ ] Prep your screen sharing demo
- [ ] Have code ready to show

### Day Of Interview
- [ ] Start with elevator pitch
- [ ] Offer to walk through code
- [ ] Reference specific sections: "Let me show you the exact code..."
- [ ] Emphasize production practices and business value

---

## 💡 Best Practices from This Project

✅ **Always Include:** Metadata tracking (timestamps, file names)  
✅ **Always Validate:** Every data quality issue with business confirmation  
✅ **Always Document:** Code comments explain "why", not just "what"  
✅ **Always Archive:** Processed files to prevent reprocessing  
✅ **Always Join:** Reference data to ensure consistency  
✅ **Always Aggregate:** At the appropriate grain for analytics  
✅ **Always Merge:** With parent/master data for single source of truth  

---

## 🎓 Key Learnings

1. **Medallion Architecture works** - Clear separation of concerns
2. **Delta MERGE is powerful** - Handles upserts elegantly  
3. **Data quality is business logic** - Not just technical rules
4. **Production code looks different** - Archives, validation, lineage
5. **Incremental loads scale** - Much better than full reloads
6. **Good documentation pays off** - Makes code maintainable

---

## 📞 Support References

**Questions About Architecture?**
- See: README.MD, WORKSPACE_DOCUMENTATION.md

**Questions About Specific Code?**
- See: Enhanced notebooks (4,000+ lines of comments)

**Interview Preparation?**
- See: PORTFOLIO_GUIDE.md (comprehensive guide)

**How to Run This?**
- See: README.MD "How to Use This Project" section

**What to Highlight?**
- See: PORTFOLIO_GUIDE.md "Standout Points"

---

## ✨ Final Tip

**When showing code in an interview:**
1. Have it open and ready to share screen
2. Start with the overall structure (README.MD)
3. Zoom into specific sections (notebooks)
4. Explain the business value first, then technical details
5. Be ready to discuss trade-offs and alternatives

**Example:** "Here's how I handle multiple date formats [show code]. This matters because real data is messy and comes from different systems. I handle 4 formats automatically, which covers 95% of cases, and explicit NULL for the rest to catch issues."

---

**Version:** 1.0  
**Last Updated:** December 2, 2025  
**Status:** ✅ Ready for Interviews

**You've got this! 🚀**
