<div align="center">

<img src="https://img.shields.io/badge/Apache%20Spark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white" />
<img src="https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white" />
<img src="https://img.shields.io/badge/Delta%20Lake-003366?style=for-the-badge&logo=delta&logoColor=white" />
<img src="https://img.shields.io/badge/Apache%20Superset-2EA04D?style=for-the-badge&logo=apache&logoColor=white" />
<img src="https://img.shields.io/badge/GPT%20OSS%2020B-412991?style=for-the-badge&logo=openai&logoColor=white" />

<br/><br/>

```
╔══════════════════════════════════════════════════════════════════╗
║                                                                  ║
║        🚚  DELIVERY ANALYTICS PLATFORM                          ║
║            End-to-End Data Pipeline + AI Query Agent            ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

<h3>Ask anything in plain English → Get instant SQL + answers</h3>

[![Pipeline Status](https://img.shields.io/badge/Pipeline-Production%20Ready-brightgreen?style=flat-square)](.)
[![Layers](https://img.shields.io/badge/Layers-Bronze%20→%20Silver%20→%20Gold-gold?style=flat-square)](.)
[![Tables](https://img.shields.io/badge/Fact%20Rows-11%2C985-blue?style=flat-square)](.)
[![AI](https://img.shields.io/badge/AI-NL%20→%20SQL%20→%20NL-purple?style=flat-square)](.)
[![License](https://img.shields.io/badge/License-MIT-lightgrey?style=flat-square)](.)

</div>

---

## 📌 What is this?

**Delivery Analytics Platform** is a production-grade **Medallion Architecture** data pipeline built on **Databricks + Delta Lake**, covering the full lifecycle of delivery data — from raw CSVs on S3 all the way to an **AI-powered natural language query engine** that lets any business user ask questions in plain English and get real answers instantly.

> *"How many orders did driver Rohit deliver in pincode 302001 last month?"*
> → The AI writes the SQL, runs it, and replies in English. No SQL knowledge needed.

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                          SOURCE LAYER                               │
│   S3: delivery-data-mohit/                                          │
│   areas.csv │ deliveries.csv │ drivers.csv │ orders.csv             │
│   payments.csv │ users.csv │ vehicles.csv                           │
└────────────────────────────┬────────────────────────────────────────┘
                             │  Schema enforcement via delivery_ddl
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        🥉  BRONZE LAYER                             │
│   delivery_cata.bronze.*                                            │
│   Raw append-only Delta tables with _ingested_at + _source_file     │
│   No transformations — exact copy of source                         │
└────────────────────────────┬────────────────────────────────────────┘
                             │  Data Quality + SCD-2 Upserts
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        🥈  SILVER LAYER                             │
│   delivery_cata.silver.*                                            │
│   ✅ Cleaned & validated  ✅ SCD-2 history tracking                 │
│   ✅ Bad rows quarantined  ✅ Status standardised                    │
│   quarantine.bad_rows  ←  invalid records captured here             │
└───────────────┬──────────────────────────────┬──────────────────────┘
                │  Star Schema (gold_rpt)       │  Flat Table (gold_ai)
                ▼                               ▼
┌───────────────────────────┐   ┌───────────────────────────────────┐
│    🥇  GOLD RPT LAYER     │   │        🥇  GOLD AI LAYER          │
│  delivery_cata.gold_rpt   │   │    delivery_cata.gold_ai          │
│                           │   │                                   │
│  fact_delivery (11,985)   │   │  master_delivery_fact (2,007)     │
│  dim_driver   (26,203)    │   │  driver_performance_daily         │
│  dim_customer (24,915)    │   │  area_daily_summary               │
│  dim_vehicle  (25,926)    │   │  customer_order_summary           │
│  dim_area     (26,252)    │   │  payment_summary                  │
│  dim_payment   (7,644)    │   │  vehicle_utilization              │
│  dim_date        (914)    │   │                                   │
└───────────┬───────────────┘   └──────────────┬────────────────────┘
            │                                  │
            ▼                                  ▼
┌─────────────────────────┐    ┌──────────────────────────────────────┐
│  📊 datbricks           │    │     🤖 AI Query Agent (NL→SQL→NL)   │
│  Operations Dashboard   │    │                                      │
│  Revenue & Payments     │    │  User asks in English                │
│  Driver Performance     │    │     → GPT OSS 20B generates SQL      │
│                         │    │     → Spark executes on Delta Lake   │
│                         │    │     → LLM answers in English         │
└─────────────────────────┘    └──────────────────────────────────────┘
```

---

## 📁 Repository Structure

```
delivery-analytics/
│
├── 📁raw data-
     
├── 📁bronze-
     ├── 📓 bronze.ipynb                  # Bronze ingestion: S3 → Delta
     ├── 📓 delivery_ddl.ipynb            # Schema definitions (DDL)
├── 📁silver-
     ├── 📓 silver.ipynb                  # Silver cleaning + SCD-2 upserts
├── 📁gold-
     ├── 📓 gold_rpt_population.ipynb     # Gold reporting: star schema build
     ├── 📓 gold_ai.ipynb                 # Gold AI: flat tables + aggregates
├── 📁ai_solution-
     ├── 📓 ai_solution.ipynb             # NL → SQL → NL agent (GPT OSS 20B)

├── 📁Delivery Analytics Dashboard-
    └── 📄 Delivery Analytics Dashboard   # Databricks deshboard
├── 📁Superset Analytics Dashboard-
    └── 📄 superset_setup.md             # Superset connection + dashboard guide
├── 📄 ai_test_queries.md                 # Full QA test suite (60+ questions)
```

---

## 🥉 Bronze Layer — `bronze.ipynb`

**Purpose:** Land raw CSV data from S3 into Delta Lake with zero transformations.

| What it does | Detail |
|---|---|
| Reads all `*.csv` files from `s3://delivery-data-mohit/` | Dynamic loop — no hardcoded filenames |
| Enforces schema | Uses `delivery_ddl` table schemas — no type inference surprises |
| Adds metadata columns | `_ingested_at` (timestamp) + `_source_file` (filename) |
| Write mode | `append` — full audit trail retained |

**Tables created:**
`bronze.areas_raw` · `bronze.deliveries_raw` · `bronze.drivers_raw` · `bronze.orders_raw` · `bronze.payments_raw` · `bronze.users_raw` · `bronze.vehicles_raw`

---

## 🥈 Silver Layer — `silver.ipynb`

**Purpose:** Apply business rules, quarantine bad data, and maintain SCD-2 history.

### Data Quality Rules per Entity

| Entity | Validation Rules |
|---|---|
| **Areas** | Non-null area_id/name, valid 6-digit pincode, delivery_charge ≥ 0 |
| **Users** | Valid Indian mobile (starts 6-9, 10 digits), valid email regex |
| **Vehicles** | Valid type (BIKE/VAN/TRUCK/CAR/SCOOTER), valid Indian plate format, capacity 0–5000 kg |
| **Drivers** | Valid name, valid Indian phone; `ON_TRIP` status now correctly preserved |
| **Orders** | Valid status enum, order_amount > 0 and < 999,999, no future `created_at` |
| **Deliveries** | Valid status, distance 0–2000 km, `delivery_time` > `pickup_time` |
| **Payments** | Valid method (CARD/UPI/CASH/WALLET/NETBANKING), `SUCCESS` normalised → `COMPLETED` |

### SCD-2 Implementation

```python
# Every entity uses full SCD-2 upsert via _row_hash comparison
apply_scd2('drivers', 'driver_id', df_good)

# Hash mismatch → close old record (effective_end, is_current=false)
#               → insert new version (effective_start=now, is_current=true)
# Hash match   → no-op (skip unchanged rows efficiently)
```

### Quarantine
Bad rows flow to `delivery_cata.quarantine.bad_rows` with:
- `source_table` — which entity had the issue
- `raw_data` — full original row as JSON
- `error_reason` — human-readable reason string

---

## 🥇 Gold Layer — Reporting (`gold_rpt_population.ipynb`)

Classic **Star Schema** optimised for BI tools and complex aggregations.

```
                    ┌─────────────┐
                    │  dim_date   │
                    └──────┬──────┘
         ┌─────────────────┼─────────────────┐
         │                 │                 │
   ┌─────┴─────┐    ┌──────┴──────┐   ┌──────┴──────┐
   │dim_driver │    │fact_delivery│   │ dim_customer│
   └─────┬─────┘    └──────┬──────┘   └──────┬──────┘
         │                 │                 │
   ┌─────┴─────┐    ┌──────┴──────┐   ┌──────┴──────┐
   │dim_vehicle│    │  dim_area   │   │ dim_payment │
   └───────────┘    └─────────────┘   └─────────────┘
```

**Key design decisions:**
- `dim_driver` and `dim_customer` retain SCD-2 history — always join with `is_current = true`
- `dim_date` auto-generated from actual delivery date range (914 dates, 2023–2025)
- `fact_delivery` denormalises `order_status` — maps `SHIPPED` → `IN_TRANSIT` for consistency
- Area linked via driver's assigned `area_id` (not order location)
- Payment is optional (LEFT JOIN) — some deliveries have no payment record

---

## 🥇 Gold Layer — AI (`gold_ai.ipynb`)

Pre-aggregated flat tables optimised for fast lookups and the AI query agent.

| Table | Rows | Purpose |
|---|---|---|
| `master_delivery_fact` | ~2,007 | Fully flat pre-joined table for simple NL queries |
| `driver_performance_daily` | — | Per-driver daily KPIs: success rate, avg time, distance |
| `area_daily_summary` | — | Per-area daily metrics: revenue, unique drivers/customers |
| `customer_order_summary` | — | Lifetime customer stats: total spent, preferred payment |
| `payment_summary` | — | Daily payment method breakdown by area |
| `vehicle_utilization` | — | Per-vehicle utilization: trips, distance, failure rate |

---

## 🤖 AI Query Agent — `ai_solution.ipynb`

**The centerpiece.** A full **NL → SQL → NL** pipeline powered by **GPT OSS 20B** running on Databricks Model Serving.

### How it works

```
User question (plain English)
        │
        ▼
┌───────────────────┐
│   nl_to_sql()     │  Sends question + full schema context to GPT OSS 20B
│                   │  Model returns a Spark SQL query
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│  extract_sql()    │  Parses model output — handles reasoning blocks,
│                   │  prose contamination, missing code fences
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│  validate_sql()   │  Blocks INSERT/UPDATE/DROP/DELETE — read-only safety
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│  spark.sql(sql)   │  Executes on Delta Lake — real data, real results
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│  result_to_nl()   │  Sends raw result JSON back to LLM for plain-English answer
└────────┬──────────┘
         │
         ▼
Plain English answer → User
```

### Schema Prompt Strategy

The agent uses a carefully engineered **schema context** with:
- Table choice rules (`master_delivery_fact` for simple, `fact_delivery + dims` for complex)
- Exact join patterns with `is_current = true` guards
- Date filtering templates (relative + absolute)
- Pincode-as-STRING reminders
- Few-shot examples covering easy to hard queries

### Query Rules (baked into system prompt)

```
✅ Driver names → always use LIKE:  LOWER(driver_name) LIKE LOWER('%Rohit%')
✅ Pincodes → always STRING:        pincode = '302001'  (not 302001)
✅ Dim joins → always guard:        AND dr.is_current = true
✅ Date filter → use order_date:    NOT DATE(pickup_time)
✅ Simple query → master_delivery_fact (fast, flat)
✅ Complex query → fact_delivery + dims (full star schema)
```

### Interactive UI

Two modes available inside the notebook:

**Mode 1 — Query Panel** (example buttons + textarea)
```
┌─────────────────────────────────────────┐
│  Delivery Analytics — AI Query Agent    │
│  Powered by GPT OSS 20B                 │
├─────────────────────────────────────────┤
│  Example questions: [btn] [btn] [btn]   │
│                                         │
│  Your question: [____________________]  │
│  [Ask]  [Clear]                         │
│                                         │
│  ✅ 42 orders were assigned to driver   │
│     Rohit across all pincodes.          │
└─────────────────────────────────────────┘
```

**Mode 2 — Chat Interface** (WhatsApp-style bubble chat)
```
                 "Which driver delivered most orders?"
                                                    [user bubble →]
[← bot bubble]  "Driver Arjun Singh delivered the
                 most orders with 87 deliveries."
```

---

## 📊 Superset Dashboards — `superset_setup`

Three production dashboards over `gold_rpt` + `gold_ai`:

### Dashboard 1 — Operations Overview
- Total deliveries by status (Pie)
- Orders over time, by status (Time-series Line)
- Revenue by city (Bar)
- Top 10 drivers by delivery count (Horizontal Bar)
- Failure rate by vehicle type (Bar)

### Dashboard 2 — Revenue & Payments
- Revenue by payment method (Pie)
- Payment status breakdown
- Monthly revenue trend (Area Chart)
- Revenue lost from failed/refunded payments (Table)

### Dashboard 3 — Driver & Area Performance
- Driver success rate table (with avg distance)
- Average delivery time by vehicle type
- Orders by pincode — top 20 (Table)
- Weekend vs Weekday split (Big Number)

**Virtual dataset** `vw_delivery_summary` pre-joins all 7 tables so chart authors never write JOINs manually.

---

## 🧪 QA Test Suite — `ai_test_queries.md`

60+ questions across 6 complexity levels to validate the AI agent end-to-end.

| Level | Category | # Questions |
|---|---|---|
| L1 | Simple lookups — single table, no joins | 10 |
| L2 | Filtered aggregations — WHERE clauses | 10 |
| L3 | Multi-table joins — fact + 1-2 dims | 10 |
| L4 | Complex aggregations & rankings | 10 |
| L5 | Edge cases & ambiguity traps | 10 |
| L6 | Business intelligence — multi-hop reasoning | 10 |

**Quick smoke test (run first):**
```
1. How many deliveries are in the system?
2. Which driver delivered the most orders?
3. What is the total revenue from UPI payments?
4. How many deliveries failed this year?
5. How many orders were placed in pincode 302001?
```

---

## ⚙️ Setup & Configuration

### Prerequisites
- Databricks workspace with Unity Catalog enabled
- SQL Warehouse (for Superset connection)
- Cluster with Delta Lake + PySpark
- GPT OSS 20B model endpoint deployed on Databricks Model Serving
- S3 bucket: `s3://delivery-data-mohit/` with 7 CSV files

### Execution Order

```bash
# Step 1 — Define schemas
Run: delivery_ddl.ipynb

# Step 2 — Ingest raw data
Run: bronze.ipynb

# Step 3 — Clean & validate
Run: silver.ipynb

# Step 4 — Build reporting layer
Run: gold_rpt_population.ipynb

# Step 5 — Build AI layer
Run: gold_ai.ipynb

# Step 6 — Start querying
Run: ai_solution.ipynb  →  ask("your question here")

# Step 7 — Connect Superset (optional)
Follow: superset_setup.md
```

### Catalog Structure

```
delivery_cata/
├── delivery_ddl/       ← Schema reference tables
├── bronze/             ← Raw Delta tables (append-only)
├── silver/             ← Cleaned + SCD-2 history
├── quarantine/         ← Bad rows with error reasons
├── gold_rpt/           ← Star schema for BI/Superset
└── gold_ai/            ← Flat + aggregated tables for AI agent
```

---

## 🔑 Key Gotchas

| Issue | Correct approach |
|---|---|
| `pincode` is STRING | `pincode = '302001'` — never `pincode = 302001` |
| Driver name search | `LOWER(driver_name) LIKE LOWER('%Rohit%')` — not exact match |
| Dim table joins | Always include `AND dr.is_current = true` |
| `SUCCESS` payments | Normalised to `COMPLETED` in silver — use `COMPLETED` in queries |
| `ON_TRIP` drivers | Now correctly preserved in silver (was previously quarantined) |
| `date_sk` filtering | Never filter on `date_sk` directly — use `full_date`, `year`, `month` |
| `master_delivery_fact` | Only ~2,007 rows — use `fact_delivery + dims` for trend analysis |
| `delivery_charge = 9999` | Data quality artifact — filter `delivery_charge < 500` in revenue charts |

---

## 📈 Data Volumes at a Glance

```
Bronze ingestion:  7 tables from S3
Silver (current):  areas ~26K | users ~25K | vehicles ~26K | 
                   drivers ~26K | orders ~? | deliveries ~12K | payments ~8K
Gold RPT:          fact_delivery = 11,985 rows | dim_driver = 26,203 | 
                   dim_customer = 24,915 | dim_vehicle = 25,926 | 
                   dim_area = 26,252 | dim_payment = 7,644 | dim_date = 914
Gold AI:           master_delivery_fact = 2,007 rows
Quarantine:        Bad rows captured with full audit trail
```

---

<div align="center">

**Built with ❤️ on Databricks · Delta Lake · Apache Spark**

*NL → SQL → NL powered by GPT OSS 20B*

</div>
