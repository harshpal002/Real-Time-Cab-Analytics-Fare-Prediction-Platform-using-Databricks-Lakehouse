# 🚕 GoodCabs — End-to-End Data Pipeline

A production-style data engineering project that builds a full **Medallion Architecture** pipeline for GoodCabs, a cab service operating across 10+ Indian cities. Raw trip data is ingested from AWS S3, cleaned and validated through Bronze → Silver → Gold layers, and delivered as city-specific analytics views.

---

## 🏗️ Architecture Overview

```
AWS S3 (Raw CSVs)
       │
       ▼
  ┌─────────────┐
  │   BRONZE    │  Raw ingestion — Auto Loader (streaming) + Materialized Views
  └─────────────┘
       │
       ▼
  ┌─────────────┐
  │   SILVER    │  Cleaned, validated, CDC upserts via SCD Type 1
  └─────────────┘
       │
       ▼
  ┌─────────────┐
  │    GOLD     │  Aggregated fact views per city for analytics
  └─────────────┘
```

---

## 📁 Project Structure

```
goodcabs-data-pipeline/
├── bronze/
│   ├── city.py          # Materialized view — batch ingestion of city dimension
│   └── trips.py         # Streaming table — Auto Loader ingestion of trip data
├── silver/
│   ├── city.py          # Cleaned city dimension
│   ├── trips.py         # Validated trips with CDC upsert (SCD Type 1)
│   └── calendar.py      # Date dimension with Indian holidays
├── gold/
│   ├── trips_gold.sql         # Core fact view joining trips + city + calendar
│   ├── trips_chandigarh.sql
│   ├── trips_coimbatore.sql
│   ├── trips_indore.sql
│   ├── trips_jaipur.sql
│   ├── trips_kochi.sql
│   ├── trips_lucknow.sql
│   ├── trips_mysore.sql
│   ├── trips_surat.sql
│   ├── trips_vadodara.sql
│   └── trips_visakhapatnam.sql
└── README.md
```

---

## ⚙️ Tech Stack

| Tool | Purpose |
|---|---|
| **Databricks** | Pipeline orchestration and compute |
| **Lakeflow / Spark Declarative Pipelines** | Pipeline definition (`@dp.table`, `@dp.materialized_view`) |
| **AWS S3** | Raw data source |
| **Delta Lake** | Storage format with ACID transactions and CDF |
| **Auto Loader** | Incremental streaming ingestion (`cloudFiles`) |
| **PySpark** | Data transformation |
| **SQL** | Gold layer view definitions |

---

## 🔄 Pipeline Walkthrough

### Bronze Layer
- **`city.py`** — Reads city CSV data from S3 as a **materialized view**. Adds `file_name` and `ingest_datetime` audit columns. Schema inference with `PERMISSIVE` mode to handle corrupt records.
- **`trips.py`** — Uses **Auto Loader** (`cloudFiles`) to stream trip CSVs incrementally. Handles schema evolution with `rescue` mode. Renames problematic column `distance_travelled(km)`.

### Silver Layer
- **`city.py`** — Selects and renames columns from bronze. Adds `silver_processed_timestamp`.
- **`trips.py`** — Applies **data quality expectations** (valid date, rating ranges). Uses `dp.create_auto_cdc_flow` with **SCD Type 1** to upsert into `silver.trips` keyed on `trip_id`.
- **`calendar.py`** — Generates a full date dimension using `sequence()`. Includes weekday/weekend flags and Indian national holidays (Republic Day, Independence Day, Gandhi Jayanti).

### Gold Layer
- **`trips_gold.sql`** — Joins `silver.trips` + `silver.city` + `silver.calendar` into a single `fact_trips` view enriched with date and city attributes.
- **City views** — One view per city, filtered from `fact_trips`, delivered to regional managers.

---

## 🚀 How to Run

1. Upload all files to your Databricks workspace
2. Create a **Lakeflow Declarative Pipeline** and attach the Bronze, Silver, and Gold files
3. Set pipeline parameters:
   - `start_date` — 
   - `end_date` —
4. Trigger a full pipeline run

---

## 🎯 Key Concepts Demonstrated

- Medallion Architecture (Bronze / Silver / Gold)
- Streaming ingestion with Auto Loader
- Change Data Capture (CDC) with SCD Type 1
- Data quality enforcement with `@dp.expect`
- Delta Lake features: Change Data Feed, Auto Optimize, Auto Compact
- Audit columns for data lineage tracking

---

## 👤 Author

Built by **Harsh** as a learning project to prepare for data engineering interviews.
