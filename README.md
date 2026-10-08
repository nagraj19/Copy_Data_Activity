# Microsoft Fabric – Copy Activity

## Overview

Microsoft Fabric Data Factory provides the **Copy Data activity** to move data from a source system into a destination such as a Fabric Lakehouse or Warehouse.

In this project, Copy Activity is used as part of the **Bronze layer ingestion process**.

The objective is to take source data, configure the source and destination, map the columns, select the appropriate update method, and load the data into the Bronze layer.

---

# Copy Activity – End-to-End Flow

The Copy Data process follows these steps:

```text
Choose Data
     ↓
Source Data
     ↓
Destination
     ↓
Update Method
     ↓
Map to Destination
     ↓
Review and Save
     ↓
Loading Data
```

---

# 1. Choose Data

The first step is to select the data that needs to be ingested.

The **Choose Data** step allows us to select the source data that will be copied into Fabric.

The source can be different types of systems, including:

* Database
* CSV files
* Excel files
* Parquet files
* Cloud storage
* APIs
* Other supported data sources

In this project, the selected source data is used for ingestion into the Bronze layer.

### Screenshot

![Choose Data](Choose%20Data.png)

---

# 2. Source Data

After selecting the source, Fabric displays the source configuration and allows us to verify the data that will be copied.

At this stage, we can confirm:

* Source connection
* Source dataset
* Source table/file
* Available columns
* Data structure

The source is kept as close to the original structure as possible because the Bronze layer is intended to preserve raw or minimally transformed data.

### Screenshot

![Source Data](Source%20Data.png)

---

# 3. Destination

The next step is to configure where the source data will be stored.

For this project, the destination is the **Fabric Lakehouse**, which is used as the storage layer for the Bronze data.

Conceptually:

```text
Source
   │
   │ Copy Activity
   ▼
Fabric Lakehouse
   │
   ▼
Bronze Layer
```

The destination configuration determines:

* Target workspace
* Target Lakehouse
* Target folder/table
* Destination format
* Target location

### Screenshot

![Destination](Destination.png)

---

# 4. Update Method

The update method determines how the data should be loaded into the destination.

Common approaches include:

* Full load
* Incremental load
* Append
* Replace

For the current implementation, the pipeline uses a full-load style process where existing target data can be removed before the new data is copied.

The current pipeline follows:

```text
Existing Bronze Data
        ↓
     Delete
        ↓
   Copy New Data
        ↓
  Bronze Lakehouse
```

This is suitable for datasets where a complete refresh is required.

For larger production datasets, an **incremental loading strategy** can be implemented using techniques such as:

* Watermark columns
* Modified date
* Change Data Capture (CDC)
* MERGE / UPSERT

### Screenshot

![Update Method](Update%20Method.png)

---

# 5. Map to Destination

The **Map to Destination** step is used to map the source columns to the destination columns.

For example:

```text
Source Column          Destination Column
------------------------------------------
OrderID          →     OrderID
CustomerID       →     CustomerID
OrderDate        →     OrderDate
Amount           →     Amount
ProductID        →     ProductID
```

Column mapping is important because the source and destination structures may not always be identical.

It also allows us to verify that the correct source fields are being written to the correct destination fields.

### Screenshot

![Map to Destination](Map%20to%20Destination.png)

---

# 6. Review and Save

Before executing the Copy Activity, the configuration can be reviewed.

This step allows us to verify:

* Source configuration
* Destination configuration
* Update method
* Column mapping
* Target location
* Overall copy configuration

After reviewing the configuration, the activity can be saved.

### Screenshot

![Review and Save](Review%20and%20save.png)

---

# 7. Loading Data

After the configuration is saved and the pipeline is executed, Fabric starts moving the source data into the configured destination.

The Copy Activity performs the data movement:

```text
Source
   │
   │
   │ Copy Activity
   ▼
Bronze Lakehouse
```

During execution, the pipeline provides information about the loading process, such as:

* Activity status
* Records processed
* Data transferred
* Execution duration
* Errors, if any

### Screenshot

![Loading Data](Loading%20Data.png)

---

# Complete Copy Activity Flow

The complete process implemented in this project is:

```text
                SOURCE DATA
                     │
                     ▼
              ┌─────────────┐
              │ Choose Data │
              └──────┬──────┘
                     │
                     ▼
              ┌─────────────┐
              │ Source Data │
              └──────┬──────┘
                     │
                     ▼
              ┌─────────────┐
              │ Destination │
              └──────┬──────┘
                     │
                     ▼
              ┌─────────────┐
              │Update Method│
              └──────┬──────┘
                     │
                     ▼
           ┌───────────────────┐
           │ Map to Destination│
           └─────────┬─────────┘
                     │
                     ▼
           ┌───────────────────┐
           │ Review and Save   │
           └─────────┬─────────┘
                     │
                     ▼
             ┌──────────────┐
             │ Loading Data │
             └──────┬───────┘
                    │
                    ▼
              BRONZE LAKEHOUSE
```

---

# Why Use Copy Activity?

Copy Activity is useful for **data ingestion and movement**.

It allows us to move data from source systems into Fabric storage without requiring us to manually write code for every source.

For example:

```text
Oracle
   ↓
Copy Activity
   ↓
Lakehouse

SQL Server
   ↓
Copy Activity
   ↓
Lakehouse

CSV
   ↓
Copy Activity
   ↓
Lakehouse
```

The transformation logic can then be handled separately in the Silver and Gold layers.

---

# Copy Activity vs Transformation

It is important to understand that **Copy Activity and transformation are different responsibilities**.

### Copy Activity

Primarily responsible for:

```text
Source
   ↓
Move Data
   ↓
Destination
```

### Transformation

Responsible for things such as:

* Removing duplicates
* Handling null values
* Changing data types
* Applying business rules
* Joining datasets
* Aggregating data

For example:

```text
Bronze
   ↓
PySpark / SQL
   ↓
Silver
```

This separation makes the Medallion Architecture easier to manage.

---

# How Copy Activity Fits into the Medallion Architecture

In this project, Copy Activity is part of the **Bronze ingestion layer**.

```text
                    SOURCE
                      │
                      ▼
              FABRIC PIPELINE
                      │
                      ▼
                COPY ACTIVITY
                      │
                      ▼
              ┌──────────────┐
              │    BRONZE    │
              │   Lakehouse  │
              └──────┬───────┘
                     │
                     ▼
              SILVER NOTEBOOK
                     │
                     ▼
              ┌──────────────┐
              │    SILVER    │
              │   Lakehouse  │
              └──────┬───────┘
                     │
                     ▼
                GOLD NOTEBOOK
                     │
                     ▼
              ┌──────────────┐
              │     GOLD     │
              │   Lakehouse  │
              └──────┬───────┘
                     │
                     ▼
                  POWER BI
```

---

# Key Concepts Learned

Through this Copy Activity implementation, the following Microsoft Fabric concepts are demonstrated:

* Fabric Data Factory
* Pipelines
* Copy Activity
* Source connections
* Destination configuration
* Lakehouse
* Bronze layer
* Column mapping
* Full-load ingestion
* Data movement
* Pipeline execution
* Medallion Architecture

---

# Important Int
