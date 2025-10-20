---

# 🧠 Data Engineering ETL Pipeline — *CSV → PySpark → AWS S3 → Airflow*

## 📋 Project Overview

This project demonstrates an **end-to-end data engineering pipeline** that

1. Ingests raw CSV data from a local or external source
2. Cleans and transforms it using **PySpark**
3. Writes processed data to an **AWS S3 data lake** in Parquet format
4. Orchestrates and schedules the workflow using **Apache Airflow**

The goal is to simulate a **production-ready ETL workflow** for analytics and reporting use-cases.

---

## 🧱 Architecture

```
                ┌────────────────┐
                │   Raw CSVs     │
                │ (Local / API)  │
                └──────┬─────────┘
                       │
                       ▼
                ┌────────────────┐
                │   PySpark ETL  │
                │ (Transform/Clean) │
                └──────┬─────────┘
                       │
                       ▼
                ┌────────────────┐
                │  AWS S3 Bucket │
                │ (Processed Data)│
                └──────┬─────────┘
                       │
                       ▼
                ┌────────────────┐
                │  Airflow DAG   │
                │(Schedule + Logs)│
                └────────────────┘
```

---

## ⚙️ Tech Stack

| Layer                | Technology            |
| -------------------- | --------------------- |
| Language             | Python 3.x            |
| Processing           | PySpark               |
| Orchestration        | Apache Airflow        |
| Storage              | AWS S3                |
| Optional Warehouse   | AWS Redshift / Athena |
| Logging & Monitoring | Airflow UI            |

---

## 📂 Project Structure

```
data-engineering-etl/
│
├── dags/
│   └── etl_pipeline_dag.py           # Airflow DAG definition
│
├── scripts/
│   ├── extract.py                    # Reads raw CSV
│   ├── transform_pyspark.py          # Cleans/transforms using PySpark
│   ├── load_to_s3.py                 # Uploads to AWS S3
│
├── data/
│   └── raw_data.csv                  # Sample input
│
├── config/
│   └── aws_config.json               # S3 bucket & IAM configs
│
├── requirements.txt                  # Dependencies
├── README.md                         # Project documentation
└── architecture.png                  # Diagram (optional)
```

---

## 🚀 Workflow Steps

### **1. Extraction**

* Read raw data (`raw_data.csv`) from a local folder or API endpoint.
* Validate schema and handle missing values.

```python
df_raw = spark.read.csv("data/raw_data.csv", header=True, inferSchema=True)
```

---

### **2. Transformation (PySpark)**

* Clean nulls, standardize column names, and derive new columns.
* Convert data types and apply business logic.

```python
df_clean = (df_raw
    .withColumnRenamed("Order Date", "order_date")
    .withColumn("year", year("order_date"))
    .dropDuplicates())
```

---

### **3. Load to AWS S3**

* Write final data as **Parquet** to your target S3 bucket.

```python
df_clean.write.mode("overwrite").parquet("s3a://your-bucket-name/processed/orders/")
```

* Validate upload using AWS Console or `boto3` list objects.

---

### **4. Orchestration (Airflow DAG)**

Define the full ETL flow in `etl_pipeline_dag.py`:

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime
from scripts.extract import extract_data
from scripts.transform_pyspark import transform_data
from scripts.load_to_s3 import load_data

with DAG("pyspark_s3_etl", start_date=datetime(2025, 10, 1),
         schedule_interval="@daily", catchup=False) as dag:

    extract = PythonOperator(task_id="extract", python_callable=extract_data)
    transform = PythonOperator(task_id="transform", python_callable=transform_data)
    load = PythonOperator(task_id="load", python_callable=load_data)

    extract >> transform >> load
```

Airflow handles:

* Daily scheduling
* Retries on failure
* Log visualization via UI

---

## 🧪 Sample Use Case

You can simulate with an **E-commerce Sales Dataset** containing:

* `Order_ID`, `Product_Category`, `Quantity`, `Price`, `Order_Date`
* Output analytics like total sales per category and year.

---

## 📊 Expected Output

* Transformed Parquet files in S3 path:
  `s3://your-bucket-name/processed/orders/`
* Example analytics table (via Athena or Redshift):

| Year | Category    | Total_Sales |
| ---- | ----------- | ----------- |
| 2023 | Clothing    | 1.2M        |
| 2023 | Electronics | 3.4M        |

---

## 🧰 Setup Instructions

### 1. Clone Repo & Install Dependencies

```bash
git clone https://github.com/yourusername/data-engineering-etl.git
cd data-engineering-etl
pip install -r requirements.txt
```

### 2. Configure AWS Credentials

Use an IAM user with `AmazonS3FullAccess`:

```bash
aws configure
```

### 3. Run Locally (Optional)

```bash
python scripts/transform_pyspark.py
```

### 4. Run via Airflow

Start Airflow services:

```bash
airflow db init
airflow webserver -p 8080
airflow scheduler
```

Access DAG on [localhost:8080](http://localhost:8080) and trigger manually.

---

## 📈 Future Enhancements

* Integrate AWS Glue crawler for schema detection
* Add data quality checks (Great Expectations / Deequ)
* Migrate output to Redshift / Snowflake for analytics
* Add Kafka streaming layer for real-time data ingestion

---

## 🧾 Author

**Abhishek**
Data Engineer in training ⚡ | AWS + PySpark + Airflow + DSA

