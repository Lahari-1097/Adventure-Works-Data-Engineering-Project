# End-to-End Azure Data Engineering Project (ADF + ADLS + Databricks + Synapse + Spark)

## Overview
This project demonstrates a complete **End-to-End Data Engineering Pipeline on Microsoft Azure** using modern cloud data engineering tools. The pipeline ingests raw data from a source dataset, stores it in **Azure Data Lake Storage Gen2**, transforms it using **Azure Databricks and Apache Spark**, and performs analytics using **Azure Synapse Analytics**. The final curated data can be consumed by **Power BI or SQL queries for reporting and analytics**.

This implementation follows the **Medallion Architecture (Bronze → Silver → Gold)** which is widely used in modern lakehouse architectures.

The project also demonstrates real-world Azure Data Engineering practices that are commonly used in enterprise data platforms and often discussed in **Azure Data Engineer interviews**.

---

# Architecture

The pipeline architecture used in this project is illustrated below.

Source Dataset  
│  
▼  
Azure Data Factory (Data Ingestion)  
│  
▼  
Azure Data Lake Storage Gen2  
│  
├── Bronze Layer (Raw Data)  
│  
▼  
Azure Databricks + Apache Spark  
│  
├── Silver Layer (Cleaned & Transformed Data)  
│  
▼  
Azure Synapse Analytics  
│  
├── Gold Layer (Analytics & Data Warehouse)  
│  
▼  
Power BI / SQL Queries / Dashboards  

---

# Technologies Used

Azure Data Factory – Data ingestion and pipeline orchestration  
Azure Data Lake Storage Gen2 – Storage for raw and processed data  
Azure Databricks – Distributed data transformation  
Apache Spark / PySpark – Big data processing  
Azure Synapse Analytics – Data warehouse and analytics engine  
Azure SQL Serverless – Querying data directly from Data Lake  
Power BI – Business intelligence and dashboards  

---

# Dataset

This project uses the **AdventureWorks dataset**, which simulates a retail company's transactional sales system.

Dataset Source  
https://www.kaggle.com/datasets/ukvet/adventureworks

The dataset contains multiple relational tables including:

Customers  
Products  
Product Categories  
Product Subcategories  
Calendar  
Territories  
Returns  
Sales (2015, 2016, 2017)

---

# Project Layers

## Bronze Layer – Raw Data

The Bronze layer stores **raw ingested data exactly as it arrives from the source system**.

Data ingestion is performed using **Azure Data Factory pipelines** and stored in **Azure Data Lake Storage Gen2**.

Example Bronze structure:

bronze/  
├── AdventureWorks_Calendar  
├── AdventureWorks_Customers  
├── AdventureWorks_Product_Categories  
├── Product_Subcategories  
├── AdventureWorks_Products  
├── AdventureWorks_Returns  
├── AdventureWorks_Sales_2015  
├── AdventureWorks_Sales_2016  
├── AdventureWorks_Sales_2017  
└── AdventureWorks_Territories  

Azure Data Factory pipelines perform:

• Data ingestion  
• File movement into Azure Data Lake  
• Workflow orchestration  
• Scheduling and automation  

---

# Silver Layer – Data Transformation

The Silver layer contains **cleaned and transformed datasets**.

Transformations are implemented using **Azure Databricks with PySpark**.

## Calendar Table Transformation

Extract month and year from the Date column.

from pyspark.sql.functions import month, year, col

df_calendar = df_calendar \
    .withColumn("Month", month(col("Date"))) \
    .withColumn("Year", year(col("Date")))

---

## Customers Table Transformation

Create a full name column.

from pyspark.sql.functions import concat_ws, col

df_customers = df_customers.withColumn(
    "FullName",
    concat_ws(" ", col("Prefix"), col("FirstName"), col("LastName"))
)

---

## Products Table Transformation

Clean SKU and simplify product name.

from pyspark.sql.functions import split, col

df_products = df_products \
    .withColumn("ProductSKU", split(col("ProductSKU"), "-")[0]) \
    .withColumn("ProductName", split(col("ProductName"), " ")[0])

---

## Sales Table Transformation

Transform sales data using PySpark.

from pyspark.sql.functions import to_timestamp, regexp_replace, col

df_sales = df_sales \
    .withColumn("StockDate", to_timestamp(col("StockDate"))) \
    .withColumn("OrderNumber", regexp_replace(col("OrderNumber"), "S", "T")) \
    .withColumn("multiply", col("OrderLineItem") * col("OrderQuantity"))

---

## Writing Data to the Silver Layer

Transformed datasets are written to the Silver layer using **Parquet format**.

df_sales.write.format("parquet") \
    .mode("append") \
    .option("path", "abfss://silver@<storage-account>.dfs.core.windows.net/AdventureWorks_Sales") \
    .save()

---

# Gold Layer – Analytics with Azure Synapse

The Gold layer contains **aggregated analytical datasets** optimized for reporting and business insights.

Azure Synapse Analytics allows querying data directly from the Data Lake using **serverless SQL pools**.

## Querying Data with OPENROWSET

SELECT *
FROM OPENROWSET(
    BULK 'https://<storage-account>.dfs.core.windows.net/silver/AdventureWorks_Sales/',
    FORMAT='PARQUET'
) AS sales

---

# Creating External Tables

External tables allow SQL queries directly on files stored in the data lake.

CREATE EXTERNAL TABLE gold.sales
(
    OrderDate DATE,
    ProductKey INT,
    OrderQuantity INT
)
WITH (
    LOCATION = 'silver/AdventureWorks_Sales/',
    DATA_SOURCE = gold_datasource,
    FILE_FORMAT = parquet_format
);

---

# Example Sales Analysis

Example analytical query performed on the sales dataset.

SELECT
    OrderDate,
    COUNT(OrderNumber) AS total_orders
FROM sales
GROUP BY OrderDate
ORDER BY OrderDate;

This produces a **daily sales trend analysis**.

---

# Power BI Integration

The curated Gold layer can be connected to **Power BI dashboards** to generate business insights such as:

• Sales performance trends  
• Product category analysis  
• Customer purchase behavior  
• Return rate analysis  
• Territory-based sales performance  

---

# Project Folder Structure

azure-data-engineering-project  
│  
├── notebooks  
│   └── databricks_transformations.py  
│  
├── data  
│   └── bronze_files  
│  
├── pipelines  
│   └── adf_pipeline.json  
│  
├── synapse  
│   └── sql_queries.sql  
│  
└── README.md  

---

# Key Data Engineering Concepts Demonstrated

This project demonstrates several important Azure data engineering concepts:

• Azure Medallion Architecture  
• Data ingestion using Azure Data Factory  
• Data Lake storage with ADLS Gen2  
• Distributed transformations using Apache Spark  
• PySpark data engineering workflows  
• Data warehousing with Azure Synapse  
• Serverless SQL queries  
• External tables in Synapse  
• Lakehouse architecture  
• Integration with Power BI  

---

# Azure Data Engineer Interview Topics Covered

This project helps demonstrate and practice concepts commonly asked in Azure data engineering interviews:

• ETL vs ELT pipeline design  
• Data Lake architecture  
• Azure Data Factory pipelines  
• Apache Spark transformations  
• Data partitioning strategies  
• Synapse serverless SQL pools  
• External table creation  
• Data warehouse modeling  
• Lakehouse architecture patterns  

---

# Conclusion

This project demonstrates how to build a **modern Azure Data Engineering pipeline** using Azure-native services and the Medallion architecture. By integrating Azure Data Factory, Azure Data Lake Storage, Databricks, Apache Spark, and Azure Synapse Analytics, we can create scalable big data solutions capable of handling large volumes of data and delivering valuable analytical insights for business decision-making.
