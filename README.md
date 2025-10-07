# YouTube Trending Data Pipeline on AWS

This project is end-to-end data engineering pipeline built on AWS to process and analyze YouTube trending video data. The main idea was to take raw JSON and CSV files from Kaggle, clean and transform them, and then build an analytical layer and dashboard to understand what makes videos trend across different categories and regions.

---

## Project Overview

The dataset I used comes from Kaggle’s **YouTube Trending Videos** data. It includes daily trending videos for multiple countries, along with their metadata and engagement metrics. I set up a full pipeline that starts with uploading the raw data to S3, processing it with AWS Lambda and Glue, transforming it into Parquet format, and finally visualizing insights in QuickSight.

The goal was to automate the entire process—so that whenever new data comes in, it gets cleaned, structured, and made ready for analysis without any manual steps.

---

## Architecture

Here’s a quick look at the flow I built:

```
Kaggle Data → S3 (Raw Layer)
            → Lambda (JSON Cleaning)
            → S3 (Cleansed Layer)
            → Glue ETL (Schema Mapping & Partitioning)
            → S3 (Analytical Layer)
            → Athena (Query Engine)
            → QuickSight (Visualization)
```

---

## Dataset Details

The dataset has two parts:

* **Category JSON**: Contains the metadata for different video categories.
* **Trending CSVs**: Contain daily trending videos and their engagement stats like views, likes, comments, etc. for multiple regions (Canada, US, Germany, India, and more).

I used both these files together to clean, join, and prepare the data for analysis.

---

## AWS Services Used

* **S3** – to store raw, cleaned, and analytical data
* **Lambda** – to clean and convert JSON files automatically on upload
* **Glue Crawlers** – to infer schema and register tables in the Data Catalog
* **Glue ETL Jobs** – to transform CSV data, fix schemas, and partition by region
* **Athena** – to join and query datasets
* **QuickSight** – to build dashboards and visualize insights

---

## Lambda Function

I wrote a Lambda function that gets triggered whenever a JSON file is uploaded to S3.
It reads the file, flattens it using `pandas` and `awswrangler`, converts the `id` column to a numeric type (to match with CSV schema), and writes it back to S3 in Parquet format.

```python
if 'id' in df_step_1.columns:
    df_step_1['id'] = pd.to_numeric(df_step_1['id'], errors='coerce')
    df_step_1['id'] = df_step_1['id'].astype('Int64')
```

This step ensures the category metadata is clean and query-ready.

---

## Glue ETL

For the CSV files, I created a Glue ETL job that:

* Reads the raw CSV data from S3
* Maps the schema (e.g., converts `category_id` to bigint)
* Drops null fields and partitions data by region
* Writes the cleaned data back to S3 in Parquet format

This transformed data is the foundation for analytics and is much more efficient to query.

---

## Analytical Layer and Dashboard

After cleaning both JSON and CSV files, I used Athena to join them on `category_id` and created an analytical table. I then connected this table to QuickSight to build dashboards showing insights like:

* Top categories by views and likes
* Category performance across different regions
* Engagement trends over time

---

## How to Run

1. Download the dataset from Kaggle and upload it to the raw S3 bucket.
2. Deploy the Lambda function to clean JSON files automatically.
3. Run Glue Crawlers to detect schemas and create tables in the Glue Catalog.
4. Execute the Glue ETL job to clean and partition the CSV data.
5. Use Athena to run queries and join datasets.
6. Connect QuickSight to Athena and build your dashboards.

---

## What I Learned

* Converting data to **Parquet** early on makes queries much faster and cheaper compared to CSV/JSON.
* Schema mismatches (like string vs bigint) can break queries if not handled at the ETL stage.
* Automating transformations with Lambda saves a lot of manual work.
* Structuring data into **raw**, **cleansed**, and **analytical** layers gives the pipeline a clear and scalable structure.

---

## Tech Stack

* **Languages**: Python, SQL, PySpark
* **AWS**: S3, Lambda, Glue (Crawlers + Jobs), Athena, QuickSight
* **Libraries**: pandas, awswrangler
