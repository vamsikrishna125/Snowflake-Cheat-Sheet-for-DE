```markdown
## 🔄 Data Pipeline Features

### Stored Procedures
**What**: Encapsulated SQL and programming logic.
**Languages**: JavaScript, Java, Python, SQL
**When**:
- Complex business logic
- Reusable operations
- Transaction management

**How**:
```sql
-- JavaScript stored procedure
CREATE OR REPLACE PROCEDURE process_orders(order_date STRING)
    RETURNS STRING
    LANGUAGE JAVASCRIPT
    EXECUTE AS CALLER
AS
$$
try {
    var sql_command = `
        INSERT INTO processed_orders
        SELECT * FROM raw_orders
        WHERE date(order_date) = '${ORDER_DATE}'
    `;
    snowflake.execute({sqlText: sql_command});
    return "Success";
} catch (err) {
    return "Failed: " + err;
}
$$;

-- Python stored procedure
CREATE OR REPLACE PROCEDURE calculate_metrics()
    RETURNS STRING
    LANGUAGE PYTHON
    RUNTIME_VERSION = '3.8'
    PACKAGES = ('pandas', 'scipy')
    HANDLER = 'run'
AS
$$
def run(snowpark_session):
    df = snowpark_session.table('raw_data')
    # Complex calculations
    return "Metrics calculated"
$$;
```

### External Functions
**What**: Integration with external API services
**Why**: Extend Snowflake capabilities with external services
**When**:
- API integrations
- Custom calculations
- Third-party services

**How**:
```sql
-- Create API integration
CREATE OR REPLACE API INTEGRATION ml_api_integration
    API_PROVIDER = aws_api_gateway
    API_AWS_ROLE_ARN = 'arn:aws:iam::123456789012:role/snowflake-api-role'
    ENABLED = true
    API_ALLOWED_PREFIXES = ('https://api-endpoint.execute-api.region.amazonaws.com/');

-- Create external function
CREATE OR REPLACE EXTERNAL FUNCTION
    predict_sentiment(text STRING)
    RETURNS VARIANT
    API_INTEGRATION = ml_api_integration
    AS 'https://api-endpoint/predict';

-- Use external function
SELECT predict_sentiment(comment_text) 
FROM customer_feedback;
```

## 🤖 Machine Learning Integration

### Snowpark ML
**What**: Native machine learning capabilities
**Why**: Build and deploy ML models directly in Snowflake
**When**:
- Predictive analytics
- Feature engineering
- Model deployment

**How**:
```python
from snowflake.snowpark.ml import LogisticRegression
from snowflake.snowpark.functions import col

def train_model(session):
    # Load data
    df = session.table('TRAINING_DATA')
    
    # Prepare features
    feature_cols = ['feature1', 'feature2', 'feature3']
    label_col = 'target'
    
    # Train model
    model = LogisticRegression()
    model.fit(df, label_col=label_col, input_cols=feature_cols)
    
    # Save model
    model.save('models/lr_model')
    
    return "Model trained successfully"
```

### Feature Engineering
**What**: Creating and managing ML features
**When**:
- Data preparation
- Model development
- Feature store implementation

**How**:
```sql
-- Create feature view
CREATE OR REPLACE VIEW customer_features AS
SELECT 
    customer_id,
    -- Recency
    DATEDIFF('day', MAX(order_date), CURRENT_DATE()) as days_since_last_order,
    -- Frequency
    COUNT(DISTINCT order_id) as total_orders,
    -- Monetary
    SUM(order_amount) as total_spent,
    AVG(order_amount) as avg_order_value
FROM orders
GROUP BY customer_id;

-- Feature store table
CREATE TABLE feature_store (
    feature_name STRING,
    feature_value VARIANT,
    computed_at TIMESTAMP_LTZ,
    valid_from TIMESTAMP_LTZ,
    valid_to TIMESTAMP_LTZ
);
```

## 📊 Data Quality & Testing

### Data Quality Checks
**What**: Automated data validation
**When**:
- Data ingestion
- Transform validation
- Quality assurance

**How**:
```sql
-- Create quality check procedure
CREATE OR REPLACE PROCEDURE validate_data()
RETURNS VARIANT
LANGUAGE SQL
AS
$$
DECLARE
    results VARIANT;
BEGIN
    -- Run various checks
    CREATE TEMPORARY TABLE quality_results AS
    SELECT 
        'Null Check' as check_name,
        SUM(CASE WHEN important_column IS NULL THEN 1 ELSE 0 END) as failures
    FROM my_table
    UNION ALL
    SELECT 
        'Range Check',
        SUM(CASE WHEN amount < 0 THEN 1 ELSE 0 END)
    FROM my_table;
    
    -- Collect results
    SELECT OBJECT_AGG(check_name, failures)
    INTO results
    FROM quality_results;
    
    RETURN results;
END;
$$;
```

### Unit Testing
**What**: Testing database objects and procedures
**When**:
- Development
- CI/CD pipelines
- Code validation

**How**:
```sql
-- Test framework
CREATE OR REPLACE PROCEDURE run_tests()
RETURNS VARIANT
LANGUAGE JAVASCRIPT
AS
$$
const tests = [
    {
        name: "Test Valid Input",
        sql: `CALL process_data('valid_input')`,
        expected: "SUCCESS"
    },
    {
        name: "Test Invalid Input",
        sql: `CALL process_data(NULL)`,
        expected: "ERROR"
    }
];

let results = [];

for (let test of tests) {
    try {
        let result = snowflake.execute({sqlText: test.sql});
        let passed = result.next().getColumnValue(1) === test.expected;
        results.push({
            test: test.name,
            passed: passed
        });
    } catch (err) {
        results.push({
            test: test.name,
            passed: false,
            error: err.message
        });
    }
}

return results;
$$;
```

## 🔗 Integration Patterns

### Microservices Integration
**What**: Patterns for microservices architecture
**When**:
- Distributed systems
- Event-driven architecture
- Service integration

**How**:
```sql
-- Event table
CREATE TABLE event_log (
    event_id VARCHAR,
    event_type VARCHAR,
    payload VARIANT,
    created_at TIMESTAMP_LTZ DEFAULT CURRENT_TIMESTAMP(),
    processed_at TIMESTAMP_LTZ
);

-- Event processing
CREATE TASK process_events
    WAREHOUSE = compute_wh
    SCHEDULE = '1 minute'
WHEN
    SYSTEM$STREAM_HAS_DATA('event_stream')
AS
CALL process_event_batch();
```

### CDC Integration
**What**: Change Data Capture patterns
**When**:
- Real-time sync
- Data replication
- Event sourcing

**How**:
```sql
-- Setup CDC
CREATE STREAM orders_changes ON TABLE orders;

-- CDC processing task
CREATE TASK process_changes
    WAREHOUSE = compute_wh
    SCHEDULE = '1 minute'
AS
INSERT INTO order_events (
    SELECT 
        order_id,
        METADATA$ACTION as change_type,
        METADATA$ISUPDATE as is_update,
        CURRENT_TIMESTAMP() as processed_at
    FROM orders_changes
);
```
