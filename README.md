# Databricks Medallion Architecture Pipeline

## Project Overview

This project implements a Medallion Architecture using Databricks, AWS S3, Unity Catalog, Delta Lake, Auto Loader, and Databricks Workflows.

The pipeline ingests raw CSV files from an AWS S3 Landing Bucket and processes them through Bronze, Silver, and Gold layers using incremental processing patterns.



# Architecture


Landing S3
    ↓
Bronze Layer
    ↓
Silver Layer
    ↓
Gold Layer




# Technologies Used

* Databricks
* Apache Spark (PySpark)
* Delta Lake
* Unity Catalog
* AWS S3
* Databricks Auto Loader
* Databricks Workflows
* Delta MERGE
* Incremental Watermark Processing


# Data Sources

The landing bucket contains the following source datasets:


orders
customers
order_items
products


Landing Path:


s3://de-case-study-landing/


---

# Bronze Layer

## Purpose

The Bronze layer stores raw ingested data with minimal transformations.

## Ingestion Method

Databricks Auto Loader (cloudFiles)

## Features

* Incremental file ingestion
* Schema evolution support
* Checkpointing
* Metadata capture
* Delta format storage

## Bronze Tables


bronze.orders_ext
bronze.customers_ext
bronze.order_items_ext
bronze.products_ext


## Additional Metadata Columns


source_system
ingestion_batch_id
ingestion_timestamp
source_file


## Checkpoint Location

s3://de-case-study-checkpoints/




# Silver Layer

## Purpose

The Silver layer performs data cleansing, standardization, deduplication, and incremental upserts.

## Features

* Data type standardization
* Column standardization
* Data quality checks
* Deduplication
* Incremental processing
* Delta MERGE

## Silver Tables


silver.orders
silver.customers
silver.order_items
silver.products


## Incremental Logic

Silver reads only new Bronze records using watermark-based processing.

Watermark Table:


metadata.pipeline_watermark


## Deduplication Keys

Orders:

order_id


Customers:


customer_id


Order Items:


order_id
order_item_id


Products:


product_id




# Gold Layer

## Purpose

The Gold layer provides business-ready analytical datasets.

## Gold Tables

### Fact Table


gold.fact_sales


Contains:

* Order Information
* Customer Information
* Product Information
* Revenue Metrics

### Aggregated Tables


gold.daily_sales_summary
gold.customer_sales_summary
gold.product_sales_summary
gold.order_status_summary


## Business Metrics

Examples:

* Daily Revenue
* Sales by Product
* Sales by Customer
* Order Status Analysis
* Freight Cost Analysis



# Incremental Processing Strategy

## Bronze

Uses:


Auto Loader Checkpoints


Only new files are processed.



## Silver

Uses:


metadata.pipeline_watermark


Only new Bronze records are processed.


## Gold

Uses:


metadata.pipeline_watermark


Only changed Silver records are processed.



# Workflow Orchestration

Databricks Workflow:


create_update_bronze_layer
        ↓
create_update_silver_layer
        ↓
create_update_golden_layer


Trigger Type:

File Arrival Trigger


Landing Path:


s3://de-case-study-landing/



# Repository Structure


databricks_case_study/
│
├── create_update_bronze_layer
├── create_update_silver_layer
├── create_update_golden_layer
│
└── README.md


# Data Governance

Implemented using:

* Unity Catalog
* Managed Schemas
* Delta Lake ACID Transactions
* Schema Enforcement
* Audit Metadata Columns





# End-to-End Flow


S3 Landing
    ↓
Auto Loader
    ↓
Bronze External Delta Tables
    ↓
Silver Incremental MERGE
    ↓
Gold Analytical Tables
    ↓
Reporting / BI Consumption

