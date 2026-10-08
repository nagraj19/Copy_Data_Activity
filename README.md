# Microsoft Fabric Copy Activity – Data Loading into Lakehouse

## Overview

This section demonstrates how **Copy Activity in Microsoft Fabric Data Factory** can be used to load source data into a **Microsoft Fabric Lakehouse**.

For the initial implementation, I use a **Full Load** approach to load the complete sample dataset from the source into the destination Lakehouse.

The loading process can be implemented using different strategies depending on the data volume, frequency of changes, and business requirements.

The main loading strategies covered are:

* Full Load
* Incremental Load
* Overwrite / Full Refresh
* Incremental Load + MERGE

---

# Basic Architecture

```text
┌──────────────────────┐
│     Source Data     │
│   Sample Dataset    │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Microsoft Fabric     │
│    Data Factory      │
│                      │
│    Copy Activity     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│   Fabric Lakehouse   │
│                      │
│   Destination Table  │
└──────────────────────┘
```

---

# Technologies Used

* Microsoft Fabric
* Fabric Data Factory
* Copy Activity
* Fabric Lakehouse
* Delta Tables
* SQL
* PySpark

---

# Current Implementation – Full Load

## What is Full Load?

A **Full Load** means that the complete dataset from the source is extracted and loaded into the destination.

For example, if the source contains 100,000 records, the Copy Activity reads the complete 100,000 records and loads them into the destination Lakehouse table.

### Full Load Flow

```text
Source Table
     │
     ▼
Copy Activity
     │
     ▼
Fabric Lakehouse
     │
     ▼
Destination Table
```

---

# Full Load Implementation

For the current sample-data implementation, I use Copy Activity to perform the initial load.

## Step 1 – Configure Source

The source dataset/table is configured in the Copy Activity.

The source contains the sample data that needs to be loaded into the Lakehouse.

```text
Source
  │
  ├── Table
  ├── Columns
  └── Records
```

---

## Step 2 – Configure Destination

The destination is configured as a **Fabric Lakehouse**.

The required destination table is selected where the source data will be loaded.

```text
Source
   │
   ▼
Copy Activity
   │
   ▼
Fabric Lakehouse
   │
   └── Destination Table
```

---

## Step 3 – Column Mapping

The source columns are mapped to the corresponding destination columns.

For example:

| Source Column    | Destination Column |
| ---------------- | ------------------ |
| ID               | ID                 |
| Name             | Name               |
| Amount           | Amount             |
| LastModifiedDate | LastModifiedDate   |

The mapping ensures that the source data is loaded into the correct destination fields.

---

## Step 4 – Run the Pipeline

Once the source, destination, and mappings are configured, I run the pipeline.

The Copy Activity reads the complete dataset and writes the data into the Lakehouse.

```text
Pipeline
   │
   ▼
Copy Activity
   │
   ▼
Complete Dataset
   │
   ▼
Lakehouse Table
```

---

# Data Validation

After the pipeline execution is completed, I validate the destination data.

The validation includes:

* Record count comparison
* Sample record validation
* Column mapping validation
* Data type validation
* Null value checks
* Pipeline execution status

For example:

```text
Source Record Count      : 100,000
Destination Record Count : 100,000
```

If the counts and sample records match the expected results, the initial load is considered successful.

---

# When to Use Full Load

Full Load is suitable when:

* It is the initial data load.
* The source dataset is relatively small.
* The complete dataset needs to be refreshed.
* There is no reliable incremental column.
* The pipeline does not run frequently.
* Simplicity is more important than incremental processing.

---

# Why Full Load May Not Be Suitable for Production

Although Full Load is simple, it may not be efficient for large production tables.

For example:

```text
Source Table
10 Million Records
       │
       ▼
   Full Load
       │
       ▼
10 Million Records
```

If only 50,000 records changed since the previous execution, processing all 10 million records again would result in unnecessary data movement.

Therefore, for production workloads, I would evaluate whether an **Incremental Load** strategy is more appropriate.

---

# Alternative 1 – Incremental Load

## What is Incremental Load?

Incremental Load means that only the **new or changed records** are loaded into the destination instead of processing the complete source dataset.

A common approach is to use a column such as:

```text
LastModifiedDate
```

or another reliable change-tracking mechanism.

---

# Incremental Load Flow

```text
┌──────────────────────┐
│      Source Data     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Identify New/Changed │
│       Records        │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│    Copy Activity     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Lakehouse Staging    │
└──────────────────────┘
```

---

# Example – Incremental Column

Suppose the source table contains:

| Column           | Description             |
| ---------------- | ----------------------- |
| ID               | Business Key            |
| Name             | Customer Name           |
| Amount           | Transaction Amount      |
| LastModifiedDate | Last Modified Timestamp |

The first pipeline execution performs a Full Load.

After that, the pipeline stores the timestamp of the last successful load.

For the next execution, only records satisfying the following condition are retrieved:

```sql
LastModifiedDate > LastSuccessfulLoadTime
```

### Example

Previous successful load:

```text
LastSuccessfulLoadTime
2026-10-07 23:59:59
```

New records:

```text
ID    LastModifiedDate
101   2026-10-08 08:10:00
102   2026-10-08 09:15:00
103   2026-10-08 10:20:00
```

These records are identified as new or changed records and are processed by the pipeline.

---

# Advantages of Incremental Load

Incremental loading can provide:

* Better performance
* Faster pipeline execution
* Less data movement
* Lower load on the source system
* Better scalability
* More efficient processing of large tables

---

# When to Use Incremental Load

I would consider Incremental Load when:

* The source table is large.
* The pipeline runs frequently.
* Only a small percentage of records change between executions.
* A reliable watermark column is available.
* The source supports CDC or another change-tracking mechanism.

Examples of change-detection mechanisms include:

* `LastModifiedDate`
* Timestamp columns
* Increasing ID
* Change Data Capture (CDC)
* Change Tracking

---

# Alternative 2 – Overwrite / Full Refresh

## What is Overwrite?

An Overwrite or Full Refresh strategy means that the complete current dataset is loaded from the source and the existing destination data is replaced.

### Flow

```text
Source
  │
  ▼
Complete Dataset
  │
  ▼
Copy Activity
  │
  ▼
Overwrite / Replace
  │
  ▼
Lakehouse Table
```

---

# When to Use Overwrite

I would consider Overwrite when:

* The table is relatively small.
* The source represents the complete current state.
* Historical versions are not required.
* The business requires a complete refresh.
* Simplicity is preferred over incremental processing.

### Example

Suppose I have a reference table containing:

```text
50,000 Records
```

If the table is small enough to refresh efficiently, I could simply reload the complete dataset rather than implementing incremental processing.

This makes the pipeline simpler and easier to maintain.

---

# Alternative 3 – Incremental Load + MERGE

For large transactional tables where both **new records and updates** need to be handled, I can use an Incremental Load combined with a `MERGE` operation.

### Architecture

```text
┌──────────────────────┐
│      Source Data     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Identify Incremental │
│   New/Changed Data   │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│    Copy Activity     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│   Staging Table      │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│       MERGE          │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│   Target Lakehouse   │
│       Table          │
└──────────────────────┘
```

---

# How Incremental + MERGE Works

The process can be divided into four steps.

### Step 1 – Identify Changes

First, I identify new and modified records from the source.

```text
Source
  │
  └── New Records
  └── Modified Records
```

### Step 2 – Load to Staging

The incremental records are loaded into a staging table.

```text
Incremental Data
       │
       ▼
Staging Table
```

### Step 3 – Compare with Target

The staging records are compared with the target table using a business key such as:

```text
ID
```

### Step 4 – MERGE

The `MERGE` operation handles both scenarios.

```text
Existing ID
     │
     ▼
  UPDATE

New ID
     │
     ▼
  INSERT
```

---

# MERGE Example

Conceptually, the MERGE operation can look like:

```sql
MERGE INTO Target AS T
USING Staging AS S
ON T.ID = S.ID

WHEN MATCHED THEN
    UPDATE SET
        T.Name = S.Name,
        T.Amount = S.Amount,
        T.LastModifiedDate = S.LastModifiedDate

WHEN NOT MATCHED THEN
    INSERT (
        ID,
        Name,
        Amount,
        LastModifiedDate
    )
    VALUES (
        S.ID,
        S.Name,
        S.Amount,
        S.LastModifiedDate
    );
```

The result is:

```text
New Record
    │
    └── INSERT

Existing Record
    │
    └── UPDATE
```

---

# Example – Why MERGE Is Useful

Suppose the target Lakehouse table contains:

```text
1,000,000 Records
```

During today's execution, only:

```text
10,000 Records
```

are new or modified.

Instead of reloading all 1 million records, I can process the 10,000 incremental records and use `MERGE` to update or insert them into the target.

This is much more suitable for large transactional datasets.

---

# Full Load vs Incremental vs Overwrite vs MERGE

| Strategy                | Description                            | Suitable For               |
| ----------------------- | -------------------------------------- | -------------------------- |
| **Full Load**           | Loads complete dataset                 | Initial load               |
| **Incremental Load**    | Loads only new/changed records         | Large tables               |
| **Overwrite**           | Replaces complete destination data     | Small/reference tables     |
| **Incremental + MERGE** | Inserts new + updates existing records | Large transactional tables |
| **Incremental + SCD**   | Maintains historical changes           | Historical reporting       |

---

# Decision Framework

The loading strategy should be selected based on the data and business requirements.

```text
                    Data Loading Requirement
                             │
                             ▼
                  Is this the initial load?
                     /               \
                   YES                NO
                    │                  │
                    ▼                  ▼
                Full Load       Is the table small?
                                  /          \
                                YES          NO
                                 │            │
                                 ▼            ▼
                              Overwrite   Is change tracking
                                           available?
                                           /        \
                                         NO         YES
                                         │           │
                                         ▼           ▼
                                      Full Load   Incremental
                                                   Load
                                                     │
                                                     ▼
                                            Are updates required?
                                               /          \
                                             NO           YES
                                              │             │
                                              ▼             ▼
                                        Incremental    Incremental
                                           Load         + MERGE
```

---

# How I Would Choose the Strategy

| Requirement                          | Strategy                     |
| ------------------------------------ | ---------------------------- |
| Initial data ingestion               | **Full Load**                |
| Small reference table                | **Overwrite / Full Refresh** |
| Large table with only new records    | **Incremental Load**         |
| Large table with inserts and updates | **Incremental + MERGE**      |
| Historical record tracking           | **Incremental + SCD**        |
| No reliable change-detection column  | **Full Load / Full Refresh** |

---

# Current Project Approach

For my current sample-data implementation, I use:

```text
Source
   │
   ▼
Copy Activity
   │
   ▼
Full Load
   │
   ▼
Fabric Lakehouse
```

I use Full Load because this is the **initial implementation** and I want to establish the complete dataset in the destination Lakehouse.

After loading the data, I validate the destination.

---

# Production Approach

For a production implementation, I would first analyze:

* Data volume
* Data change frequency
* Pipeline execution frequency
* Availability of a watermark column
* CDC/change-tracking capabilities
* Whether updates are required
* Historical data requirements
* Performance requirements

Based on these requirements, I would select the appropriate strategy.

For example:

```text
Initial Load
     │
     ▼
  Full Load
     │
     ▼
Initial Target
     │
     ▼
Daily Pipeline
     │
     ▼
Incremental Load
     │
     ▼
Staging
     │
     ▼
MERGE
     │
     ▼
Updated Target
```

---

# Interview Explanation

If the interviewer asks:

### "How are you loading data into the Lakehouse?"

I would explain:

> "In my current implementation, I use Copy Activity in Microsoft Fabric Data Factory to load sample source data into a Fabric Lakehouse. For the initial load, I use a Full Load approach, where the complete source dataset is copied into the destination table. After the pipeline execution, I validate the record count and sample records between the source and destination."

### "Would you always use Full Load?"

I would say:

> "No. Full Load is suitable for my initial load, but in production I would choose the loading strategy based on data volume, change frequency, and business requirements."

### "What would you use for a large table?"

I would say:

> "If the source has a reliable watermark such as LastModifiedDate, I would use Incremental Load so that I process only new or changed records. If I also need to update existing records, I would load the incremental data into a staging table and then use MERGE to perform inserts and updates against the target table."

### "When would you use Overwrite?"

I would say:

> "If the table is relatively small and the source represents the complete current state, I could use an Overwrite or Full Refresh approach. It is simpler because I don't need to maintain incremental logic."

---

# Key Learning

The main learning from this implementation is that **Copy Activity is not limited to one loading strategy**.

The appropriate approach depends on the characteristics of the data and the business requirement.

The overall decision can be summarized as:

**Initial Load → Full Load**

**Small Table → Overwrite / Full Refresh**

**Large Table + New/Changed Records → Incremental Load**

**Large Table + Inserts + Updates → Incremental Load + MERGE**

**Historical Requirements → Incremental Load + SCD**

Therefore, in a production environment, I would not simply choose Full Load for every pipeline execution. I would evaluate **data volume, change frequency, source capabilities, performance, and historical requirements** before selecting the appropriate loading strategy.
