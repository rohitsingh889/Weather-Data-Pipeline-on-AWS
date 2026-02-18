# 🌦 Incremental Weather Data Lake Pipeline on AWS

![Project Overview](https://github.com/rohitsingh889/--weather-lake-pipeline/blob/main/PICS/AWS%20pipeline%20main%20architecture.png)

This project implements a production-style **batch data engineering pipeline** that ingests historical weather data from the **Open-Meteo Archive API**, stores immutable raw data in Amazon S3, performs distributed transformations using **AWS Glue (PySpark)**, generates analytics-ready datasets, and enables SQL-based querying via **AWS Glue Data Catalog and Amazon Athena** — fully orchestrated by **Apache Airflow in a Dockerized local environment**.

**Data Source API:** https://open-meteo.com/

---

## 🚀 Project Objective

The goal of this pipeline is to demonstrate **real-world cloud data engineering patterns**, including:

✔ Workflow orchestration  
✔ API-driven ingestion  
✔ Multi-layer data lake architecture  
✔ Distributed Spark transformations  
✔ Incremental processing strategy  
✔ Data quality validation  
✔ Metadata-driven analytics  
✔ BI consumption workflows  

---

## 🏗 High-Level Architecture

**Pipeline Flow**

Open-Meteo Archive API  
→ Python Extraction Layer  
→ Amazon S3 (Bronze Layer – Raw JSON)  
→ AWS Glue (Silver Layer – PySpark Transformations + Data Quality Checks)  
→ Amazon S3 (Silver Layer – Parquet)  
→ AWS Glue (Gold Layer – Aggregations)  
→ Amazon S3 (Gold Layer – Analytics-Ready Parquet)  
→ AWS Glue Crawler  
→ AWS Glue Data Catalog  
→ Amazon Athena  
→ BI / Analytics Tools (Power BI)

---

![Project Overview](https://github.com/rohitsingh889/--weather-lake-pipeline/blob/main/PICS/pipelineinfo.png)

---

## 🧩 Technologies & Tools Used

| Category | Technology / Tool | Purpose |
|----------|-------------------|---------|
| Orchestration | Apache Airflow (Dockerized) | Workflow scheduling & dependency management |
| Containerization | Docker | Local Airflow environment isolation |
| Language | Python | API ingestion & orchestration logic |
| HTTP Client | Requests Module | REST API communication |
| AWS SDK | Boto3 | Programmatic AWS interaction |
| Cloud Storage | Amazon S3 | Data lake storage layers |
| Distributed Processing | AWS Glue (PySpark) | Spark-based ETL transformations |
| Metadata Discovery | AWS Glue Crawler | Schema inference & partition detection |
| Metadata Catalog | AWS Glue Data Catalog | Athena table definitions |
| Query Engine | Amazon Athena | SQL querying on S3 |
| BI Tool | Power BI | Analytics & visualization |
| Credential Management | AWS CLI + IAM Roles | Secure AWS authentication |

---

## 🔄 End-to-End Data Lifecycle (Step-Wise)

The pipeline follows a structured **Extract → Transform → Load → Analytics** workflow.

---

### 1️⃣ Extract Phase – API Ingestion

✔ Weather data retrieved from the Open-Meteo Archive API  
✔ Python used as the ingestion engine  
✔ REST calls executed via the `requests` module  
✔ Raw JSON responses preserved  

**Output**

→ Raw JSON stored in the Bronze layer (Amazon S3)

![S3 Bucket](https://github.com/rohitsingh889/--weather-lake-pipeline/blob/main/PICS/s3%20bucket.png)

![Bronze Layer](https://github.com/rohitsingh889/--weather-lake-pipeline/blob/main/PICS/bronze.png)

---

### 2️⃣ Transform Phase – Silver Processing (AWS Glue)

✔ Partition-specific Bronze data read  
✔ Nested arrays flattened  
✔ Timestamps parsed  
✔ Data types standardized  
✔ Duplicate records removed  
✔ Data quality checks enforced  

**Output**

→ Cleaned Parquet datasets written to the Silver layer

![Silver Layer](https://github.com/rohitsingh889/--weather-lake-pipeline/blob/main/PICS/silver.png)

---

### 3️⃣ Load Phase – Gold Aggregation (AWS Glue)

✔ Silver datasets aggregated into daily metrics  
✔ Business-friendly schema produced  
✔ Dataset optimized for analytics  

**Output**

→ Analytics-ready Parquet datasets written to the Gold layer

![Gold Layer](https://github.com/rohitsingh889/--weather-lake-pipeline/blob/main/PICS/gold.png)

---

### 4️⃣ Analytics Phase – Query & BI Consumption

✔ Glue Crawler infers schema  
✔ Glue Data Catalog stores metadata  
✔ Athena executes SQL queries  
✔ Power BI consumes Gold datasets  

**Purpose**

→ Business intelligence & reporting

---

## 📡 Data Source Layer

**Provider:** Open-Meteo Archive API  

The pipeline retrieves **historical hourly weather observations** for configured cities.

**Data Retrieved**

- temperature_2m  
- precipitation  
- windspeed_10m  
- timestamp  

The API returns nested JSON structures requiring flattening.

---

## ⚙ Extraction Layer (Python)

### Responsibilities

✔ Call Open-Meteo API using `requests`  
✔ Fetch previous day’s data dynamically  
✔ Preserve raw JSON responses  
✔ Upload directly to the S3 Bronze layer using `boto3`

All source Python scripts are stored in the **same configured DAG location inside the Dockerized Airflow environment**, ensuring consistent execution and simplified orchestration.

---

### Libraries Used

**Requests Module** – REST API communication  
**Boto3 (AWS SDK)** – Secure AWS interaction

✔ S3 PutObject operations  
✔ IAM-based authentication  
✔ No hardcoded AWS credentials  

---

## 🔐 IAM Role & Security Model

Secure access to AWS services is enforced using **IAM roles and least-privilege permissions**.

✔ Glue jobs execute using an IAM role with scoped S3 + Glue permissions  
✔ No static credentials embedded in code  
✔ Airflow interacts with AWS via configured credentials  
✔ Access controlled through AWS policy-based authorization  

Typical permissions include:

- S3 read/write access to Bronze / Silver / Gold paths  
- Glue job execution permissions  
- Glue crawler execution permissions  
- Athena query access (via catalog)

This design follows **cloud security best practices** and mirrors real production environments.

---

## 🗂 Bronze Layer – Raw Data Zone

**Storage:** Amazon S3  
**Format:** Raw JSON  

```
bronze/weather/
    city=XYZ/
        year=YYYY/
            month=MM/
                day=DD/
```

✔ Immutable storage  
✔ Replay capability  
✔ No transformations applied  

---

## 🔄 Silver Layer – Transformation & Validation Zone

**Processing Engine:** AWS Glue (PySpark)

✔ Flatten nested structures  
✔ Standardize schema  
✔ Apply data quality checks  

```
silver/weather/date=YYYY-MM-DD/
```

Bad data → Job fails intentionally.

---

## 📊 Gold Layer – Analytics Zone

**Processing Engine:** AWS Glue (Aggregation Job)

Aggregations:

- avg_temperature  
- max_temperature  
- total_precipitation  
- avg_windspeed  

```
gold/weather/date=YYYY-MM-DD/
```

✔ BI-ready datasets  
✔ Optimized for Athena  

---

## 🔁 Incremental Processing Strategy

✔ Process only `process_date`  
✔ Overwrite only target partition  
✔ Idempotent reruns  

```python
.mode("overwrite")
.option("replaceWhere", "date = 'YYYY-MM-DD'")
```

---

## ⛓ Orchestration Layer – Apache Airflow

Airflow acts as the **central control plane**, coordinating task execution and dependency management.

✔ Schedule runs  
✔ Trigger Glue jobs  
✔ Trigger Crawlers  
✔ Retry & failure handling  

---

## 🔍 Querying with Amazon Athena

✔ Serverless SQL on S3  
✔ No infrastructure management  
✔ Pay-per-query pricing  

---

## 📈 BI / Visualization Layer – Power BI

✔ Athena used as SQL backend  
✔ Gold datasets optimized for dashboards  

NOTE: BI dashboard upload pending — will be completed soon.

---

## 💰 Cost Optimization Strategy

✔ Parquet columnar format  
✔ Partitioned datasets  
✔ Reduced Athena scan costs  

---

## 👨‍💻 Author & Project Context

**Rohit Raj Singh**

Key skills demonstrated:

- Workflow orchestration with Apache Airflow  
- REST API ingestion & data lake design  
- AWS Glue distributed ETL (PySpark)  
- Incremental processing & validation  
- Athena-based analytics  
- IAM-secured AWS integration  

📬 **LinkedIn:**  
https://www.linkedin.com/in/rohit-raj-singh-3030172a4
