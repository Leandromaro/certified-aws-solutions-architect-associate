# Data Analytics

## Amazon EMR

* EMR = **Elastic MapReduce**
* EMR helps create **Hadoop clusters** to process and analyze vast amounts of data (Big Data)
* The cluster can be made of hundreds of **EC2 instances**
* Supports **Apache Spark, HBase, Presto, Flink**, and other big data frameworks
* EMR takes care of **provisioning and configuration**
* Provides **auto-scaling** and integrates with **Spot Instances**
* Use cases:

  * Big Data processing
  * Machine learning
  * Web indexing
  * Large-scale data analysis

---

## AWS Glue

* Fully managed **ETL (Extract, Transform, Load)** service
* Automates time-consuming steps of **data preparation for analytics**
* **Serverless**, pay-as-you-go, and fully managed
* Uses **Apache Spark** under the hood for many ETL workloads
* Can **crawl data sources** and identify data formats and schemas (**schema inference**)
* Can automatically generate ETL code and allows customization of Spark-based jobs
* Sources include:

  * Aurora
  * RDS
  * Redshift
  * S3
* Sinks include:

  * S3
  * Redshift
  * Other supported data stores
* **Glue Data Catalog** stores metadata about datasets, including:

  * Table definitions
  * Column names
  * Data types
  * Schema
  * Location of the data

### Mental Model

```text
Data Sources
     │
     ▼
┌──────────────┐
│ AWS Glue     │
│ ETL / Crawler│
└──────┬───────┘
       │
       ▼
Glue Data Catalog
   (Metadata)
       │
       ▼
  Data Analytics
```

---

## Amazon Athena

* **Serverless interactive query service**
* Allows you to query data stored directly in **Amazon S3 using SQL**
* No servers or infrastructure to manage
* No need to load the data into a database first
* Commonly used for **ad-hoc queries and data analysis**
* Integrates with the **AWS Glue Data Catalog**
* Supports formats such as:

  * CSV
  * JSON
  * Parquet
  * ORC
  * Avro
* Common use case: analyzing **logs, datasets, and files stored in S3**

### Mental Model

```text
       Amazon S3
      Actual Data
           │
           ▼
        Athena
        SQL Query
           │
           ▼
         Results
```

### Athena + Glue

```text
             S3
          Actual Data
              │
              ▼
       Glue Data Catalog
        Schema / Metadata
              │
              ▼
           Athena
           SQL Query
```

### Athena Cost Optimization

Athena query costs are primarily based on the **amount of data processed/scanned**.

To reduce costs:

* Use **Parquet or ORC** instead of CSV/JSON when appropriate
* **Partition** data in S3
* Avoid `SELECT *`
* Query only the columns and partitions you need

Example:

```sql
SELECT customer_id, amount
FROM sales
WHERE year = 2026
  AND month = 9;
```

---

## Athena Federated Query

* Allows Athena to query **data outside of S3**
* Uses **data source connectors** to access external data sources
* You can use SQL to query multiple data sources
* Data does **not need to be moved into S3 or another database first**
* Common sources include:

  * RDS
  * DynamoDB
  * Redshift
  * Other supported data sources

### Mental Model

```text
                     Athena
                        │
          ┌─────────────┼─────────────┐
          │             │             │
          ▼             ▼             ▼
         S3            RDS       DynamoDB
       Data          Database      Data
```

### Key Concept

**Normal Athena:**

```text
S3 → Athena → SQL
```

**Athena Federated Query:**

```text
Multiple Data Sources
        │
        ▼
      Athena
        │
       SQL
```

### SAA-C03 Exam Clue

If the question says:

> "Query data across multiple data sources using SQL without moving the data into a single database."

Think:

**Athena Federated Query**

---

# Amazon Redshift

* Fully managed **cloud data warehouse**
* Designed for **large-scale analytical workloads**
* Uses SQL
* Optimized for **OLAP (Online Analytical Processing)**
* Useful for:

  * Business intelligence (BI)
  * Data warehousing
  * Complex analytical queries
  * Dashboards and reporting
  * Large-scale data analytics
* Can be **provisioned** or **serverless**
* Unlike Athena, Redshift is designed around a **data warehouse** rather than simply querying files in S3

### Mental Model

```text
Data Sources
     │
     ▼
┌──────────────┐
│  Redshift    │
│              │
│ Data         │
│ Warehouse    │
└──────┬───────┘
       │
       ▼
 BI / Analytics
 / Reporting
```

---

## Redshift Spectrum

**Redshift Spectrum** allows Redshift to query data stored directly in **S3** without loading all of that data into Redshift.

```text
                Redshift
              ┌───────────┐
              │ Warehouse │
              └─────┬─────┘
                    │
             Redshift Spectrum
                    │
                    ▼
                   S3
            External Data
```

This allows you to combine:

* Data stored inside Redshift
* Data stored externally in S3

using SQL.

---

# Athena vs Redshift

|                       | Athena                       | Redshift                               |
| --------------------- | ---------------------------- | -------------------------------------- |
| Type                  | Serverless query service     | Data warehouse                         |
| Primary use           | Ad-hoc queries               | Data warehousing & analytics           |
| SQL                   | ✅                            | ✅                                      |
| Server management     | None                         | Managed by AWS                         |
| Data in S3            | Directly queried             | Can be queried using Spectrum          |
| Requires loading data | ❌                            | Usually                                |
| Complex analytics     | ✅                            | ✅                                      |
| BI / dashboards       | Possible                     | Common use case                        |
| Best for              | Occasional/ad-hoc S3 queries | Repeated, complex analytical workloads |

### Mental Model

**Athena:**

> "I already have data in S3 and I want to query it using SQL."

```text
S3 → Athena → SQL
```

**Redshift:**

> "I need a data warehouse for large-scale analytics."

```text
Data → Redshift → Analytics
```

**Redshift Spectrum:**

> "I have a Redshift warehouse, but I also want to query external data in S3."

```text
Redshift ──┐
           ├── SQL Query
S3 ────────┘
```

---

# Data Analytics — Quick Comparison

| Service                    | Main Purpose                | Key Concept                |
| -------------------------- | --------------------------- | -------------------------- |
| **EMR**                    | Big Data processing         | Hadoop / Spark clusters    |
| **Glue**                   | ETL & data preparation      | Serverless ETL             |
| **Glue Data Catalog**      | Metadata                    | Schema / table definitions |
| **Athena**                 | SQL queries on S3           | Serverless SQL             |
| **Athena Federated Query** | SQL across external sources | Connectors                 |
| **Redshift**               | Data warehouse              | Large-scale analytics      |
| **Redshift Spectrum**      | Query S3 from Redshift      | External S3 data           |

## SAA-C03 Memory Rules

```text
Big Data processing / Spark
        ↓
       EMR

ETL / Data preparation
        ↓
       Glue

Metadata / Schema
        ↓
Glue Data Catalog

SQL + data already in S3
        ↓
      Athena

SQL + multiple external data sources
        ↓
Athena Federated Query

Data Warehouse / BI / complex analytics
        ↓
     Redshift

Redshift + external data in S3
        ↓
Redshift Spectrum
```
