### Enhanced Snowflake Cheat Sheet for Data Engineers
---
#### **1. Modern Data Engineering Concepts**
- **Zero-Copy Cloning:**
```sql
-- Clone entire database
CREATE DATABASE dev_db CLONE prod_db;
-- Clone specific table
CREATE TABLE table_backup CLONE original_table;
```

- **Time Travel:**
```sql
-- Query data as it existed 1 hour ago
SELECT * FROM my_table AT(OFFSET => -60*60);
-- Restore deleted table
UNDROP TABLE my_table;
```

- **Data Sharing:**
```sql
-- Create share
CREATE SHARE my_share;
-- Grant usage on database
GRANT USAGE ON DATABASE my_database TO SHARE my_share;
```

#### **2. Advanced Data Loading Patterns**
- **Snowpipe with Auto-Ingest:**
```sql
CREATE PIPE my_pipe AUTO_INGEST=true AS
COPY INTO my_table
FROM @my_stage/data/
FILE_FORMAT = (TYPE = 'PARQUET');
```

- **Bulk Loading with Error Handling:**
```sql
COPY INTO my_table
FROM @my_stage
FILE_FORMAT = (TYPE = 'CSV')
ON_ERROR = CONTINUE
VALIDATION_MODE = RETURN_ERRORS;
```

#### **3. Performance Optimization Techniques**
- **Clustering Keys:**
```sql
CREATE TABLE sales (
    date_time TIMESTAMP,
    product_id INT,
    amount DECIMAL
) CLUSTER BY (date_time);

-- Reclustering
ALTER TABLE sales RECLUSTER;
```

- **Search Optimization:**
```sql
ALTER TABLE customers ADD SEARCH OPTIMIZATION;
```

#### **4. Modern Data Types and Functions**
- **Geography Data Type:**
```sql
CREATE TABLE locations (
    id INT,
    location GEOGRAPHY
);

-- Insert point
INSERT INTO locations VALUES
(1, ST_POINT(-122.35, 37.55));
```

- **Array Operations:**
```sql
-- Array functions
SELECT ARRAY_SIZE(my_array),
       ARRAY_CONTAINS(my_array, 'value'),
       ARRAY_CAT(array1, array2);
```

#### **5. Advanced Security Features**
- **Dynamic Data Masking:**
```sql
CREATE MASKING POLICY email_mask AS
(val string) RETURNS string ->
    CASE
        WHEN CURRENT_ROLE() IN ('ADMIN') THEN val
        ELSE REGEXP_REPLACE(val, 
            '^([^@]+)@(.+)$', '****@\\2')
    END;

ALTER TABLE users MODIFY COLUMN email 
SET MASKING POLICY email_mask;
```

#### **6. Snowpark for Data Engineering**
```python
# Complex ETL with Snowpark
from snowflake.snowpark.functions import col, sum

def transform_data(session):
    # Read source data
    df = session.table("raw_data")
    
    # Complex transformation
    transformed = (df
        .group_by("category")
        .agg(sum(col("amount")).alias("total_amount"))
        .filter(col("total_amount") > 1000)
    )
    
    # Write results
    transformed.write.mode("overwrite").save_as_table("aggregated_results")
```

#### **7. Task Orchestration**
```sql
-- Create task with upstream dependencies
CREATE OR REPLACE TASK child_task
    WAREHOUSE = compute_wh
    AFTER parent_task
AS
    CALL process_data();

-- Task with conditional execution
CREATE OR REPLACE TASK conditional_task
    WAREHOUSE = compute_wh
    SCHEDULE = 'USING CRON 0 9 * * * America/Los_Angeles'
    WHEN SYSTEM$STREAM_HAS_DATA('my_stream')
AS
    INSERT INTO target_table 
    SELECT * FROM source_table;
```

#### **8. Cost Optimization Techniques**
```sql
-- Resource monitoring
CREATE RESOURCE MONITOR monthly_limit
WITH 
    CREDIT_QUOTA = 1000
    FREQUENCY = MONTHLY
    START_TIMESTAMP = IMMEDIATELY
    TRIGGERS 
        ON 75 PERCENT DO NOTIFY
        ON 100 PERCENT DO SUSPEND;

-- Warehouse auto-suspension
ALTER WAREHOUSE compute_wh 
SET AUTO_SUSPEND = 60
    AUTO_RESUME = TRUE
    MIN_CLUSTER_COUNT = 1
    MAX_CLUSTER_COUNT = 3;
```

#### **9. Development Best Practices**
- Use `TRANSIENT` tables for temporary data
- Implement proper error handling in procedures
- Use staged development (DEV → QA → PROD)
- Regular performance monitoring
- Implement proper backup strategies
- Use tags for resource tracking
- Document DAG dependencies

#### **10. Monitoring and Troubleshooting**
```sql
-- Query history analysis
SELECT *
FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY())
WHERE EXECUTION_STATUS = 'FAILED'
AND START_TIME >= DATEADD(hours, -24, CURRENT_TIMESTAMP());

-- Warehouse monitoring
SELECT * 
FROM TABLE(INFORMATION_SCHEMA.WAREHOUSE_METERING_HISTORY(
    DATE_RANGE_START=>DATEADD('days',-7,CURRENT_DATE()),
    DATE_RANGE_END=>CURRENT_DATE()));
```

This enhanced version includes more practical examples and modern features that data engineers commonly use in their daily work. Would you like me to expand on any particular section?
