```markdown
# 🔥 Ultimate Snowflake Cheat Sheet for Data Engineers

A comprehensive guide explaining WHAT, WHY, WHEN, and HOW to use Snowflake features.

## 📑 Table of Contents
- [Core Concepts](#core-concepts)
- [Data Architecture](#data-architecture)
- [Storage & Compute](#storage--compute)
- [Data Loading](#data-loading)
- [Data Transformation](#data-transformation)
- [Security Features](#security-features)
- [Performance Features](#performance-features)
- [Monitoring & Optimization](#monitoring--optimization)

## 🎯 Core Concepts

### Virtual Warehouses
**What**: Compute clusters that execute SQL queries and DML operations.
**Why**: Separate compute from storage for independent scaling.
**When**: 
- Small (XS, S) - For basic querying and development
- Medium (M, L) - For ETL and moderate analytics
- Large (XL, 2XL+) - For heavy data processing

**How**:
```sql
-- Create a warehouse
CREATE WAREHOUSE etl_warehouse WITH
    WAREHOUSE_SIZE = 'LARGE'
    AUTO_SUSPEND = 300
    AUTO_RESUME = TRUE
    MIN_CLUSTER_COUNT = 1
    MAX_CLUSTER_COUNT = 3;

-- Scale up/down as needed
ALTER WAREHOUSE etl_warehouse 
SET WAREHOUSE_SIZE = 'XLARGE';
```

### Databases & Schemas
**What**: Logical containers organizing your data objects.
**Why**: Maintain clear data organization and access control.
**When**: 
- Databases: Separate different applications or environments
- Schemas: Organize related objects within a database

**How**:
```sql
-- Create hierarchy
CREATE DATABASE analytics;
CREATE SCHEMA analytics.raw;
CREATE SCHEMA analytics.transformed;

-- Set context
USE DATABASE analytics;
USE SCHEMA raw;
```

## 📦 Data Architecture

### Tables
**What**: Primary storage objects for structured data.
**Types**:
1. Permanent - Default, persisted tables
2. Transient - Temporary tables with no fail-safe
3. Temporary - Session-specific tables

**When to Use Each**:
- Permanent: Production data, critical business data
- Transient: Staging tables, intermediate results
- Temporary: Session-specific computations

**How**:
```sql
-- Permanent table
CREATE TABLE customers (
    customer_id INT,
    name STRING,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP()
);

-- Transient table
CREATE TRANSIENT TABLE staging_customers 
    LIKE customers;

-- Temporary table
CREATE TEMPORARY TABLE temp_results (
    metric STRING,
    value FLOAT
);
```

### Views
**What**: Virtual tables based on SELECT statements.
**Types**:
1. Standard Views - Regular SQL views
2. Materialized Views - Pre-computed result sets
3. Secure Views - Views with additional security

**When**:
- Standard: Simple data transformations
- Materialized: Frequently accessed aggregations
- Secure: Sensitive data access

**How**:
```sql
-- Standard view
CREATE VIEW customer_summary AS
SELECT 
    region,
    COUNT(*) as customer_count
FROM customers
GROUP BY region;

-- Materialized view
CREATE MATERIALIZED VIEW daily_sales AS
SELECT 
    DATE_TRUNC('day', sale_date) as day,
    SUM(amount) as total_sales
FROM sales
GROUP BY 1;

-- Secure view
CREATE SECURE VIEW customer_pii AS
SELECT 
    customer_id,
    CASE WHEN current_role() = 'ADMIN' 
         THEN email 
         ELSE '***' END as email
FROM customers;
```

## 🔄 Data Loading

### Stages
**What**: Locations where data files are stored for loading.
**Types**:
1. Internal - Snowflake-managed storage
2. External - Cloud storage (S3, Azure, GCS)
3. User - User-specific temporary stage

**When**:
- Internal: Small-scale data loading
- External: Large-scale or continuous loading
- User: Ad-hoc file operations

**How**:
```sql
-- Internal stage
CREATE STAGE my_internal_stage;

-- External stage (AWS S3)
CREATE STAGE my_external_stage
    URL = 's3://my-bucket/path'
    CREDENTIALS = (AWS_KEY_ID = 'xxx' 
                  AWS_SECRET_KEY = 'xxx');

-- List stage contents
LIST @my_internal_stage;
```

### Snowpipe
**What**: Continuous data ingestion service.
**Why**: Real-time or near-real-time data loading.
**When**: 
- Streaming data ingestion
- Micro-batch processing
- Event-driven architectures

**How**:
```sql
-- Create file format
CREATE FILE FORMAT csv_format
    TYPE = CSV
    FIELD_DELIMITER = ','
    SKIP_HEADER = 1;

-- Create pipe
CREATE PIPE my_snowpipe 
    AUTO_INGEST = true AS
    COPY INTO my_table
    FROM @my_stage/data/
    FILE_FORMAT = csv_format;

-- Monitor pipe status
SELECT SYSTEM$PIPE_STATUS('my_snowpipe');
```

### Bulk Loading
**What**: Large-scale data loading operations.
**Why**: Efficient loading of large datasets.
**When**: 
- Initial data loads
- Large batch processing
- Historical data migration

**How**:
```sql
-- Basic COPY command
COPY INTO my_table
FROM @my_stage/data/
FILE_FORMAT = (TYPE = 'PARQUET')
PATTERN = '.*[.]parquet'
ON_ERROR = CONTINUE
VALIDATION_MODE = RETURN_ERRORS;

-- Load with transformations
COPY INTO my_table
FROM (
    SELECT 
        $1::INT as id,
        UPPER($2::STRING) as name,
        PARSE_JSON($3) as properties
    FROM @my_stage/data/
)
FILE_FORMAT = (TYPE = 'CSV');
```
```

This is Part 1 of the enhanced cheat sheet. Would you like me to continue with the remaining sections covering:
1. Data Transformation Features
2. Security & Governance
3. Performance Optimization
4. Monitoring & Maintenance
5. Modern Features (Snowpark, Tasks, Streams)

Each section will follow the same pattern of explaining WHAT, WHY, WHEN, and HOW to use each feature.
