# Snowflake S3 Pipeline Project

## Author
Vishal Gaud

## Description
This project demonstrates a **full data pipeline** using **AWS S3** and **Snowflake**.  
It ingests raw CSV files from S3, stores them in a Snowflake table, processes active employees in a dynamic table, and exports cleaned data back to S3.

The pipeline includes:

1. **Raw Data in S3** → `Raw_Data` stage  
2. **Source Table in Snowflake** → `source_table` stores raw employee data  
3. **Pipe (`S3_pipe`)** → Automatically ingests new files from S3  
4. **Dynamic Table (`cleaned_data`)** → Processes only active employees  
5. **Export** → Cleaned CSV is written to `Cleaned_Data` S3 stage

---

## Snowflake Objects

- **Tables**: `source_table`  
- **Dynamic Tables**: `cleaned_data`  
- **Stages**: `raw_data_stage`, `cleaned_data_stage`  
- **Storage Integration**: `S3_int`  
- **Pipe**: `S3_pipe`  
- **File Format**: `CSV_format`  

---

## AWS Setup and Configuration

### 1. S3 Buckets
Create two S3 buckets (or folders inside a bucket):

- `Raw_Data` → for incoming raw CSV files  
- `Cleaned_Data` → for processed CSV files from Snowflake  

Example paths:  
- `s3://snowflakeintigration/Raw_Data/`
- `s3://snowflakeintigration/Cleaned_Data/`


### 2. IAM Role for Snowflake
- Create an **IAM Role** (e.g., `Snowflake_S3`) with the following permissions:
  - `s3:GetObject` and `s3:ListBucket` for `Raw_Data`
  - `s3:PutObject` and `s3:ListBucket` for `Cleaned_Data`
- Set **Trusted Entity** to allow Snowflake access.
- Copy the **Role ARN**, e.g.: `arn:aws:iam::293595299677:role/Snowflake_S3`


### 3. Enable S3 Event Notifications
- Configure the `Raw_Data` bucket to send **event notifications** to Snowflake whenever a new file is uploaded.  
- This allows `auto_ingest = true` in the Snowflake pipe to trigger automatically.

### 4. Snowflake Storage Integration
Create a storage integration in Snowflake to connect to AWS S3:
```sql
create or replace storage integration S3_int
type = external_stage
storage_provider = s3
enabled = true
storage_aws_role_arn = '<your-iam-role-arn>'
storage_allowed_locations = (
    's3://snowflakeintigration/Raw_Data/',
    's3://snowflakeintigration/Cleaned_Data/'
);

