-- ===================================================
-- Snowflake S3 Pipeline Project
-- Author: Vishal Gaud
-- Description: Full pipeline from S3 raw data → Snowflake table → Dynamic table → Export to cleaned S3
-- ===================================================

-- Set Role, Warehouse, Database, Schema
use role accountadmin;
use warehouse compute_wh;
use database mydb;
use schema mydb.myschema;

-- ================================
-- 1. Create Source Table
-- ================================
create or replace table source_table
(
    id INT IDENTITY(1,1) PRIMARY KEY,
    fname STRING,
    lname STRING,
    age INT,
    dob DATE,
    doj DATE,
    salary INT,
    department STRING,
    emp_status STRING,
    email STRING,
    phone STRING,
    city STRING
);

-- ================================
-- 2. Create Storage Integration
-- ================================
create or replace storage integration S3_int
type = external_stage
storage_provider = s3
enabled = true
storage_aws_role_arn = '<your-iam-role-arn>'
storage_allowed_locations = (
    's3://snowflakeintigration/Raw_Data/',
    's3://snowflakeintigration/Cleaned_Data/'
);

desc integration S3_int;

-- ================================
-- 3. Create Stages
-- ================================
create or replace stage raw_data_stage
url = 's3://snowflakeintigration/Raw_Data/'
storage_integration = S3_int;

create or replace stage cleaned_data_stage
url = 's3://snowflakeintigration/Cleaned_Data/'
storage_integration = S3_int;

-- ================================
-- 4. Create File Format
-- ================================
CREATE OR REPLACE FILE FORMAT CSV_format
TYPE = 'CSV'
FIELD_DELIMITER = ','
SKIP_HEADER = 1
FIELD_OPTIONALLY_ENCLOSED_BY = '"'
NULL_IF = ('NULL', 'null')
COMPRESSION = 'NONE';

-- ================================
-- 5. Create Pipe for Auto Ingest
-- ================================
create or replace pipe S3_pipe
auto_ingest = true
as
copy into source_table
(fname, lname, age, dob, doj, salary, department, emp_status, email, phone, city)
from @raw_data_stage
file_format = (format_name = CSV_Format);

show pipes;

-- Check pipe status
select system$pipe_status('S3_pipe');

-- ================================
-- 6. Create Dynamic Table
-- ================================
create or replace dynamic table cleaned_data
target_lag = '1 minute'
warehouse = compute_wh
as
select
    concat(fname,' ',lname) as full_name,
    age,
    doj as date_of_joining,
    salary,
    department
from
    source_table
where emp_status = 'Active';

-- ================================
-- 7. Export Cleaned Data to S3
-- ================================
copy into @cleaned_data_stage/data.csv
from cleaned_data
file_format = (format_name = CSV_format)
overwrite = true
single = true;
