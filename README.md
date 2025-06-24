# Data-Pipeline

# 🛠️ Data Engineering Pipeline: XML → Kafka → S3 → EMR → Redshift (Orchestrated with Airflow)

This project implements a production-grade data pipeline using AWS and open-source tools to ingest, validate, and load large XML datasets into Redshift for analytics.

---

## 📌 Use Case

Ingest **100GB XML files** daily from an on-prem SFTP server, parse and transform them, stream the data into Kafka, store in S3, and load into Amazon Redshift via EMR — all orchestrated by Apache Airflow.

---

## 🧱 Architecture Overview

FTP/SFTP (XML)
↓
Python Parser (XML → CSV)
↓
Kafka Producer (Amazon MSK)
↓
Kafka S3 Sink Connector
↓
Amazon S3 (Raw Layer)
↓
Apache Airflow (MWAA)
├── EMR Cluster Creation
├── Spark Step 1: File Validation
├── Spark Step 2: Content Validation
├── Spark Step 3: Truncated Stage Load
├── Spark Step 4: Main Load
└── EMR Termination
↓
Amazon Redshift (Staging → Main Table)


---

## 🔗 Components Used

| Component            | Role                                                                 |
|----------------------|----------------------------------------------------------------------|
| **SFTP (on-prem)**   | Source of large daily XML files                                      |
| **Python Agent**     | Connects to SFTP, validates via XSD, parses XML to CSV               |
| **AWS Secrets Manager** | Securely stores SFTP/Kafka credentials                          |
| **Apache Kafka (MSK)** | Streaming platform to publish parsed data                        |
| **Kafka S3 Sink Connector** | Automatically writes Kafka messages to S3                    |
| **Amazon S3**        | Raw data storage layer                                               |
| **Apache Airflow (MWAA)** | DAG orchestrates file detection, EMR steps, and retry logic     |
| **Amazon EMR**       | Spark jobs for validation and Redshift loading                      |
| **Amazon Redshift**  | Final analytics database                                             |

---

## 🧪 EMR Steps Explained

| Step                   | Description                                                                 |
|------------------------|-----------------------------------------------------------------------------|
| `file_validation.py`   | Validates schema, structure, delimiter, header rows                        |
| `content_validation.py`| Checks data types, business rules, null handling                           |
| `truncated_stage_load.py` | Truncates Redshift staging table and loads new data from S3            |
| `main_load.py`         | Merges staged data into production Redshift tables                         |

---

## 📜 Airflow DAG Logic

- **Sensor**: Waits for new file in `s3://your-bucket/raw/yyyy/mm/dd/`
- **Create EMR Cluster**: On file arrival
- **Add Steps**: All Spark jobs added to EMR
- **Watch Last Step**: Polls for success/failure
- **Terminate EMR**: Whether pass/fail (with max 3 retries)

---

## 🔐 Security

- **AWS Secrets Manager**: Stores SFTP/Kafka credentials securely
- **IAM Roles**: Used for accessing S3, EMR, and Redshift
- **VPC/Subnet isolation** for data security

---

## 🧰 Future Enhancements

- Replace EMR with AWS Glue for managed Spark
- Add AWS Step Functions for modular step control
- Integrate with Amazon QuickSight or Tableau for dashboards
- Add CloudWatch monitoring for EMR and Airflow failures

---

## 📂 Repository Structure

├── dags/
│ └── emr_orchestrated_etl.py # Airflow DAG
├── scripts/
│ ├── file_validation.py # Spark validation script
│ ├── content_validation.py
│ ├── truncated_stage_load.py
│ └── main_load.py
├── README.md



---

## ✅ Summary

This pipeline provides an end-to-end automated solution for processing XML data with full fault-tolerance, retries, monitoring, and transformation using scalable cloud-native services.

---
