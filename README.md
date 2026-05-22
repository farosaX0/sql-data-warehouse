# SQL Data Warehouse Project

A fully functional **Data Warehouse** built with SQL Server, implementing the **Medallion Architecture** (Bronze → Silver → Gold) to consolidate and transform data from two real-world source systems — CRM and ERP — into a clean, analytics-ready Star Schema.

---

## Architecture Overview

```
Source Systems          Data Warehouse                    Consume
─────────────     ──────────────────────────────     ──────────────
  CRM (CSV)   ──►  Bronze  ──►  Silver  ──►  Gold  ──►  Power BI
  ERP (CSV)                                         ──►  Ad-hoc SQL
                                                    ──►  AI & ML
```

### Layer Breakdown

| Layer  | Object Type | Load Strategy         | Transformations                                              |
|--------|-------------|----------------------|--------------------------------------------------------------|
| Bronze | Tables      | Full Load (Truncate & Insert) | None — raw data as-is                              |
| Silver | Tables      | Full Load (Truncate & Insert) | Cleansing, Standardization, Normalization, Enrichment |
| Gold   | Views       | No Load (query-time) | Integration, Aggregation, Business Logic                     |

---

## Data Model — Star Schema (Gold Layer)

```
          dim_customers
               │
fact_sales ────┤
               │
          dim_products
```

- **`gold.fact_sales`** — core transactional table (orders, quantities, prices, dates)
- **`gold.dim_customers`** — unified customer dimension (CRM + ERP joined, with surrogate keys)
- **`gold.dim_products`** — enriched product dimension (category, subcategory, product line)

---

## ETL Pipeline Details

### Bronze Layer — Raw Ingestion
- Loads CSV files from CRM and ERP source systems using `BULK INSERT`
- Encapsulated in a stored procedure (`bronze.load_bronze`) with:
  - Performance tracking (load duration per table in ms)
  - Full error handling via `TRY/CATCH`
  - Detailed print logging for monitoring

### Silver Layer — Cleansing & Transformation
- Stored procedure (`silver.load_silver`) applies:
  - **Deduplication** using `ROW_NUMBER()` — keeps the most recent customer record
  - **Date validation** — invalid or future dates set to NULL
  - **Sales recalculation** — corrects inconsistent sales values (`quantity × price`)
  - **Gender & marital status normalization** — raw codes mapped to readable values
  - **Prefix stripping & key extraction** — standardizes IDs across CRM and ERP
  - **Derived end dates** using `LEAD()` window function for product history

### Gold Layer — Business-Ready Views
- No physical load — views query Silver at runtime
- Integrates CRM and ERP customer data with smart fallback logic (`COALESCE`)
- Generates surrogate keys using `ROW_NUMBER()`
- Filters out historical products — only current records exposed

---

## Repository Structure

```
sql-data-warehouse/
├── datasets/               # Source CSV files (CRM & ERP)
│   ├── source_crm/
│   └── source_erp/
├── docs/
│   └── data_architecture.png
│   └── data_catalog.md
│   └── data_integration.png
│   └── naming_conventions.md
|
├── scripts/
│   ├── bronze/
│   │   ├── ddl_bronze.sql          # Table definitions
│   │   └── proc_load_bronze.sql    # ETL stored procedure
│   ├── silver/
│   │   ├── ddl_silver.sql          # Table definitions
│   │   └── proc_load_silver.sql    # ETL stored procedure
│   └── gold/
│       └── ddl_gold.sql            # View definitions (Star Schema)
└── tests/                  # Data quality checks
    ├── quality_checks_silver.sql
    ├── quality_checks_gold.sql
```

---

## Tech Stack

- **SQL Server** (T-SQL)
- **Medallion Architecture** (Bronze / Silver / Gold)
- **Star Schema** data modeling
- **Stored Procedures** with error handling and performance logging
- **Window Functions** — `ROW_NUMBER()`, `LEAD()`
- **draw.io** for architecture documentation

---

## How to Run

1. Clone the repository
2. Open SQL Server Management Studio (SSMS)
3. Run `scripts/bronze/ddl_bronze.sql` to create Bronze tables
4. Run `scripts/silver/ddl_silver.sql` to create Silver tables
5. Run `scripts/gold/ddl_gold.sql` to create Gold views
6. Execute `EXEC bronze.load_bronze` to load raw data
7. Execute `EXEC silver.load_silver` to transform and clean data
8. Query Gold views for analytics: `SELECT * FROM gold.fact_sales`

---

## Key Concepts Demonstrated

- Multi-source data integration (CRM + ERP)
- Data quality handling (nulls, duplicates, invalid values, inconsistent formats)
- ETL pipeline design with full error handling
- Dimensional modeling (Star Schema)
- Slowly Changing Dimension logic (product history via `LEAD()`)
- Industry-standard warehouse layering (Medallion Architecture)
