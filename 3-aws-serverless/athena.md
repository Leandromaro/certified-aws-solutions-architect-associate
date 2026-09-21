# Amazon Athena

## What is Amazon Athena?

**Amazon Athena is a serverless, interactive query service that allows you to analyze data directly in Amazon S3 using standard SQL.**

You don't need to provision or manage servers, databases, or clusters.

### Mental Model

```text
                Amazon S3
        ┌─────────────────────┐
        │ CSV                 │
        │ JSON                │
        │ Parquet             │
        │ ORC                 │
        └──────────┬──────────┘
                   │
                   │ Read data
                   ▼
           ┌───────────────┐
           │    Athena     │
           │   SQL Query   │
           └───────┬───────┘
                   │
                   ▼
                Results
```

---

## Key Characteristics

| Feature                               | Athena |
| ------------------------------------- | ------ |
| Serverless                            | ✅      |
| Queries data in S3                    | ✅      |
| Uses SQL                              | ✅      |
| Requires EC2                          | ❌      |
| Requires managing servers             | ❌      |
| Requires loading data into a database | ❌      |
| Integrates with AWS Glue Data Catalog | ✅      |
| Supports CSV                          | ✅      |
| Supports JSON                         | ✅      |
| Supports Parquet                      | ✅      |
| Supports ORC                          | ✅      |
| Supports Avro                         | ✅      |

---

## How Athena Works

Suppose you have sales data stored in S3:

```text
s3://my-bucket/sales/
    ├── sales-2025.csv
    ├── sales-2026.csv
    └── ...
```

You can define a table in Athena and query the data directly:

```sql
SELECT
    customer_id,
    SUM(amount)
FROM sales
GROUP BY customer_id;
```

Athena does **not** need to import the data into a database first.

The data remains in S3.

---

# Athena + AWS Glue

Athena is commonly used together with the **AWS Glue Data Catalog**.

```text
                Amazon S3
                   │
                   │ Data
                   ▼
          ┌─────────────────┐
          │   Glue Crawler  │
          └────────┬────────┘
                   │
                   │ Schema / Metadata
                   ▼
          ┌─────────────────┐
          │ Glue Data       │
          │ Catalog         │
          └────────┬────────┘
                   │
                   ▼
          ┌─────────────────┐
          │     Athena      │
          │   SQL Queries   │
          └─────────────────┘
```

### AWS Glue Data Catalog

The Glue Data Catalog stores **metadata about the data**, such as:

```text
Table: sales

customer_id → string
date        → date
amount      → double
```

The actual data is still stored in S3.

### Glue Crawler

A **Glue Crawler** can automatically discover data in S3 and determine its schema.

It can then create or update table definitions in the Glue Data Catalog.

---

# Athena Pricing

A key concept for the SAA exam:

> **Athena pricing is primarily based on the amount of data processed by queries.**

For example:

```sql
SELECT *
FROM huge_table;
```

If the query needs to scan a large amount of data, it can become expensive.

Therefore, you should optimize your data and queries.

---

## How to Reduce Athena Costs

### 1. Use Columnar Formats

Prefer:

```text
Parquet
ORC
```

instead of:

```text
CSV
JSON
```

Columnar formats allow Athena to read only the columns required by the query.

For example:

```sql
SELECT customer_id
FROM sales;
```

With a columnar format, Athena doesn't necessarily need to read every column in the dataset.

---

### 2. Partition Your Data

Instead of storing everything in one location:

```text
s3://bucket/sales/
```

partition the data:

```text
s3://bucket/sales/
    year=2025/
        month=01/
        month=02/
    year=2026/
        month=01/
        month=02/
```

Then query only the required partition:

```sql
SELECT *
FROM sales
WHERE year = 2026
  AND month = 01;
```

This can significantly reduce the amount of data Athena needs to scan.

---

### 3. Avoid `SELECT *`

Instead of:

```sql
SELECT *
FROM sales;
```

prefer:

```sql
SELECT customer_id, amount
FROM sales;
```

This is especially important when using columnar formats.

---

# Athena vs Other AWS Services

## Athena vs S3

They have different responsibilities:

```text
S3
→ Stores the data

Athena
→ Queries the data using SQL
```

---

## Athena vs Glue

```text
Glue Data Catalog
→ Stores metadata/schema

Athena
→ Queries the data
```

Glue can also provide ETL capabilities, while Athena is primarily a query service.

---

## Athena vs Redshift

|                       | Athena                   | Redshift                     |
| --------------------- | ------------------------ | ---------------------------- |
| Type                  | Serverless query service | Data warehouse               |
| Primary data location | S3                       | Redshift storage             |
| SQL                   | ✅                        | ✅                            |
| Server management     | None                     | Managed by AWS               |
| Best for              | Ad-hoc queries over S3   | Data warehousing / analytics |
| Data must be loaded?  | ❌                        | Typically yes                |
| Directly queries S3   | ✅                        | Can integrate with S3        |

### Mental Model

Use **Athena** when:

> "My data is already in S3 and I want to query it using SQL without managing infrastructure."

Use **Redshift** when:

> "I need a data warehouse for structured, repeated, high-performance analytics workloads."

---

# SAA-C03 Exam Tip

When you see a question describing:

* Large amounts of data
* Data stored in **S3**
* Need to run **SQL queries**
* Ad-hoc analysis
* No servers to manage
* Serverless solution

Think:

> **Amazon Athena**

### Classic Exam Scenario

**Question:**

> A company stores several terabytes of log data in Amazon S3. The company needs to run occasional SQL queries against the logs without managing any infrastructure. What AWS service should they use?

**Answer: Amazon Athena**

```text
S3
 │
 │ Data
 ▼
Athena
 │
 │ SQL
 ▼
Results
```

---

# Remember

### The 3-service mental model

```text
┌──────────────────────────┐
│          S3              │
│                          │
│     Actual Data          │
└────────────┬─────────────┘
             │
             │
             ▼
┌──────────────────────────┐
│    Glue Data Catalog     │
│                          │
│    Metadata / Schema     │
└────────────┬─────────────┘
             │
             │
             ▼
┌──────────────────────────┐
│         Athena           │
│                          │
│       SQL Queries        │
└──────────────────────────┘
```

**S3 = Data**

**Glue = Metadata**

**Athena = SQL queries**

---

## One-Line Definition

> **Amazon Athena is a serverless SQL query service that analyzes data directly in Amazon S3 without requiring you to manage infrastructure.**
