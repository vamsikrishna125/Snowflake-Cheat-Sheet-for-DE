```markdown
## 🔄 Data Transformation Features

### Streams
**What**: Change tracking mechanism for tables and views.
**Types**:
1. Standard Streams - Track DML changes
2. Append-Only Streams - Track only inserts
3. Insert-Only Streams - External table changes

**When**:
- CDC (Change Data Capture) implementations
- Incremental data processing
- Real-time data pipelines

**How**:
```sql
-- Create standard stream
CREATE STREAM orders_stream ON TABLE orders;

-- Create append-only stream
CREATE STREAM orders_stream_append 
    ON TABLE orders
    APPEND_ONLY = TRUE;

-- Consume stream
SELECT * FROM orders_stream
WHERE METADATA$ACTION = 'INSERT';

-- Create table from stream
CREATE TABLE orders_archive AS
SELECT * FROM orders_stream;
```

### Tasks
**What**: Scheduled or triggered execution of SQL commands.
**Why**: Automation of data workflows.
**Types**:
1. Scheduled Tasks - Time-based execution
2. Dependent Tasks - DAG-based execution
3. Conditional Tasks - Event-driven execution

**When**:
- Scheduled data transformations
- Automated maintenance
- Complex workflow orchestration

**How**:
```sql
-- Create scheduled task
CREATE TASK daily_summary
    WAREHOUSE = transform_wh
    SCHEDULE = 'USING CRON 0 0 * * * UTC'
AS
    INSERT INTO daily_metrics
    SELECT date_trunc('day', ts) as day,
           COUNT(*) as count
    FROM events;

-- Create dependent task
CREATE TASK child_task
    WAREHOUSE = transform_wh
    AFTER parent_task
AS
    CALL process_data();

-- Task management
ALTER TASK daily_summary RESUME;
ALTER TASK daily_summary SUSPEND;
EXECUTE TASK daily_summary;
```

## 🔒 Security & Governance

### Access Control
**What**: Multi-layered security model for data access.
**Components**:
1. Users - Individual accounts
2. Roles - Collections of privileges
3. Privileges - Specific permissions
4. Resource Monitors - Usage control

**When**:
- Setting up new environments
- Implementing security policies
- Managing data access

**How**:
```sql
-- Role hierarchy
CREATE ROLE junior_analyst;
CREATE ROLE senior_analyst;
GRANT ROLE junior_analyst TO ROLE senior_analyst;

-- User management
CREATE USER jane_doe
    PASSWORD = 'xxxx'
    DEFAULT_ROLE = junior_analyst
    MUST_CHANGE_PASSWORD = TRUE;

-- Privilege assignment
GRANT USAGE ON WAREHOUSE analyst_wh TO ROLE junior_analyst;
GRANT SELECT ON SCHEMA analytics.public TO ROLE junior_analyst;
GRANT INSERT ON TABLE reports TO ROLE senior_analyst;
```

### Data Governance
**What**: Features ensuring data quality and compliance.
**Components**:
1. Tags - Metadata management
2. Row Access Policies - Row-level security
3. Masking Policies - Column-level security

**When**:
- Implementing compliance requirements
- Managing sensitive data
- Data classification

**How**:
```sql
-- Tagging
CREATE TAG pii_level;
ALTER TAG pii_level SET MASKING_POLICY = (
    'HIGH', 'MEDIUM', 'LOW'
);

ALTER TABLE customers 
    MODIFY COLUMN email 
    SET TAG pii_level = 'HIGH';

-- Row access policy
CREATE ROW ACCESS POLICY department_rap AS
(department_id NUMBER) 
RETURNS BOOLEAN ->
    current_role() = 'ADMIN' OR
    department_id IN (
        SELECT dept_id 
        FROM user_departments
        WHERE user_id = current_user()
    );

-- Apply row access policy
ALTER TABLE employees
ADD ROW ACCESS POLICY department_rap ON (department_id);
```

## ⚡ Performance Optimization

### Clustering
**What**: Physical organization of table data.
**Why**: Improve query performance through data ordering.
**When**:
- Large tables (1TB+)
- Frequent range queries
- Common filter conditions

**How**:
```sql
-- Create clustered table
CREATE TABLE sales (
    sale_date DATE,
    store_id INT,
    amount DECIMAL
) CLUSTER BY (sale_date);

-- Analyze clustering
SELECT SYSTEM$CLUSTERING_INFORMATION('sales', '(sale_date)');

-- Recluster manually
ALTER TABLE sales RECLUSTER;
```

### Query Optimization
**What**: Techniques to improve query performance.
**Components**:
1. Query Profile
2. Result Cache
3. Partition Pruning

**When**:
- Performance tuning
- Cost optimization
- Query troubleshooting

**How**:
```sql
-- Query profiling
EXPLAIN USING TEXT
SELECT * FROM large_table
WHERE date_column BETWEEN '2023-01-01' AND '2023-12-31';

-- Search optimization
ALTER TABLE products 
ADD SEARCH OPTIMIZATION;

-- Results cache
ALTER SESSION SET USE_CACHED_RESULT = TRUE;
```

## 🔧 Modern Features

### Snowpark
**What**: Programmatic data processing framework.
**Supported Languages**: Python, Java, Scala
**When**:
- Complex transformations
- Machine learning pipelines
- Custom logic implementation

**How**:
```python
from snowflake.snowpark import Session
from snowflake.snowpark.functions import col, sum

def process_data(session: Session):
    # Read data
    df = session.table("raw_data")
    
    # Transform
    result = (df
        .filter(col("status") == "active")
        .group_by("category")
        .agg(sum("amount").alias("total"))
        .sort("total", ascending=False)
    )
    
    # Write results
    result.write.save_as_table("processed_data")
```

### Dynamic Tables
**What**: Self-maintaining tables that automatically update.
**Why**: Simplify pipeline maintenance and ensure data freshness.
**When**:
- Automated aggregations
- Maintaining derived datasets
- Real-time analytics

**How**:
```sql
-- Create dynamic table
CREATE DYNAMIC TABLE sales_summary
TARGET_LAG = '1 minute'
WAREHOUSE = compute_wh
AS
SELECT 
    date_trunc('hour', event_time) as hour,
    COUNT(*) as event_count,
    SUM(amount) as total_amount
FROM sales_events
GROUP BY 1;

-- Monitor refresh history
SELECT * 
FROM table(information_schema.dynamic_table_refresh_history());
```

## 📊 Monitoring & Optimization

### Resource Monitoring
**What**: Tools to track and control resource usage.
**Components**:
1. Query History
2. Warehouse Monitoring
3. Storage Metrics

**When**:
- Cost optimization
- Performance monitoring
- Capacity planning

**How**:
```sql
-- Query monitoring
SELECT 
    query_id,
    user_name,
    warehouse_name,
    execution_time,
    bytes_scanned
FROM table(information_schema.query_history())
WHERE execution_time > 60
ORDER BY start_time DESC;

-- Warehouse monitoring
SELECT 
    warehouse_name,
    SUM(credits_used) as total_credits,
    AVG(avg_running) as avg_concurrent_queries
FROM table(
    information_schema.warehouse_metering_history(
        dateadd('days', -7, current_date())
    )
)
GROUP BY 1;
```

## 📚 Additional Resources & Best Practices

### Development Best Practices
1. **Environment Management**
   - Use separate warehouses for ETL and BI
   - Implement proper environment separation
   - Use resource monitors for cost control

2. **Code Organization**
   - Use stored procedures for complex logic
   - Implement proper error handling
   - Document dependencies and flows

3. **Performance**
   - Choose appropriate warehouse sizes
   - Implement proper clustering
   - Use caching effectively

### Useful Functions & Commands
```sql
-- Time travel
SELECT * FROM my_table 
AT(OFFSET => -60*60);

-- Zero-copy cloning
CREATE DATABASE dev_db 
CLONE prod_db;

-- Data sampling
SELECT * FROM large_table
SAMPLE (10);

-- JSON handling
SELECT 
    parse_json(json_column):property::string,
    json_column:nested.property::number
FROM json_table;
```

## 🔍 Troubleshooting Guide

### Common Issues & Solutions
1. **Performance Issues**
   - Check warehouse sizing
   - Analyze query plans
   - Review clustering keys

2. **Loading Problems**
   - Verify file formats
   - Check stage permissions
   - Monitor Snowpipe status

3. **Security Issues**
   - Review role hierarchy
   - Check access controls
   - Verify masking policies

```
