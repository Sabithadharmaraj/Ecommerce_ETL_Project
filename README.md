# E-commerce ETL Project

An end-to-end **ETL & Data Warehousing pipeline** built in a Databricks notebook. It ingests raw e-commerce CSV data, cleans it, models it into **Star** and **Snowflake** schemas, and publishes the result to Unity Catalog tables, CSV exports, and an Oracle database.

## What This Project Does

1. **Extract** — loads 12 raw CSV datasets (customers, orders, order items, products, categories, sellers, warehouses, payments, shipments, returns, reviews, coupons) from a Unity Catalog volume.
2. **Clean** — data audit and cleaning: duplicate removal, text normalization, date parsing, and missing-value handling via reusable helper functions.
3. **Transform** — builds a dimensional model:
   * **Star schema** — one fact table (`fact_sales`) plus 7 dimensions (`dim_date`, `dim_customer`, `dim_product`, `dim_category`, `dim_seller`, `dim_warehouse`, `dim_coupon`) with surrogate keys and derived date keys.
   * **Snowflake schema** — further normalizes geography into `dim_location` and categories into parent/child `dim_category` tables.
4. **Load** — publishes the model to three targets:
   * **Unity Catalog** tables (`workspace.default.*`) via Spark.
   * **CSV exports** for both schemas (`etl_output/star_schema/`, `etl_output/snowflake_schema/`).
   * **Oracle database** (SQLAlchemy + python-oracledb, thin mode) — tables are recreated and bulk-inserted on each run.
5. **Validate** — row-count reconciliation, surrogate-key completeness checks, and orphan-record checks.

## Schema Tables

| Table | Type | Rows |
| --- | --- | --- |
| fact_sales | Fact | 65,000 |
| dim_customer | Dimension (star) | 15,000 |
| dim_product | Dimension (star) | 5,000 |
| dim_date | Dimension (star) | 2,178 |
| dim_seller | Dimension (star) | 800 |
| dim_coupon | Dimension (star) | 500 |
| dim_category | Dimension (star) | 84 |
| dim_warehouse | Dimension (star) | 60 |
| dim_location | Dimension (snowflake) | derived |
| dim_customer / dim_seller / dim_warehouse / dim_category / dim_product (snowflake variants) | Dimensions (snowflake) | derived |

## Tech Stack

* **Databricks** (notebook, serverless compute)
* **Python / pandas** — cleaning and modeling
* **Apache Spark** — Unity Catalog table writes
* **SQLAlchemy + python-oracledb** — Oracle export (thin mode, no client install)
* **Unity Catalog** — governed table storage

## How to Run

1. Open `E-com_ETL_Project.ipynb` in Databricks.
2. Run the cells in order: load → clean → dimensions → fact → exports.
3. Before the Oracle export cell, run `%pip install sqlalchemy oracledb` (the install cell is included in the notebook).
4. The Oracle connection block uses **placeholder credentials** (`your_oracle_username` / `your_oracle_password` / `Hostname`). Replace them with your own values — or better, load them from [Databricks secrets](https://docs.databricks.com/en/security/secrets/index.html) — before running.


## Repository Structure

```
Ecommerce_ETL_Project/
├── E-com_ETL_Project.ipynb   # Full ETL pipeline notebook
└── README.md
```
