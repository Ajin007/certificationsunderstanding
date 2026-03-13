# Oracle Concepts: Analytic Platform, Data Quality Operators, JSON Materialized View

## 1. Oracle Analytic Functions (Analytic Platform)

Analytic functions in Oracle Database perform calculations across a set
of rows related to the current row while still returning individual
rows.
Oracle has strengthened its Analytic Platform in Oracle 23c and 23ai by introducing more powerful features for large-scale data processing, real-time analytics, and enhanced machine learning (ML) capabilities. The platform integrates with big data, AI, and cloud-native analytics tools.

Key Enhancements:

Autonomous Analytics: Oracle Autonomous Database now supports automatic scaling, tuning, and optimization for analytical workloads, which helps in running large-scale analytics without manual intervention.

In-Memory Column Store: Oracle's In-Memory Columnar Store has been enhanced, allowing faster query processing for analytics queries (e.g., aggregations and complex joins). This results in real-time processing and analytics, especially when working with large datasets.

Machine Learning and Data Science:

Oracle Machine Learning (OML) is now more integrated with the database, allowing you to run machine learning models directly inside the database, minimizing data transfer costs and enhancing performance.

Automated Data Preprocessing: New features to automatically handle missing data, outliers, and feature scaling, which makes it easier to work with data for analytics and machine learning.

Seamless Integration with Big Data: Oracle has improved the integration with Hadoop, NoSQL, and cloud storage, enabling cross-platform querying and analytics, especially for big data use cases.

### Syntax

``` sql
analytic_function (expression)
OVER (
     PARTITION BY column
     ORDER BY column
)
```

-   **PARTITION BY** → divides rows into groups
-   **ORDER BY** → defines calculation order

### Example Table: EMPLOYEES

  EMP_ID   NAME    DEPT   SALARY
  -------- ------- ------ --------
  101      Amit    IT     50000
  102      Neha    IT     60000
  103      Raj     HR     40000
  104      Priya   HR     45000
### Example -- Employee type
~~~
CREATE TABLE employees (
    EMP_ID NUMBER PRIMARY KEY, 
    NAME VARCHAR2(100), 
    DEPT VARCHAR2(50), 
    SALARY NUMBER
);
INSERT INTO employees (EMP_ID, NAME, DEPT, SALARY) 
VALUES (101, 'Amit', 'IT', 50000);

INSERT INTO employees (EMP_ID, NAME, DEPT, SALARY) 
VALUES (102, 'Neha', 'IT', 60000);

INSERT INTO employees (EMP_ID, NAME, DEPT, SALARY) 
VALUES (103, 'Raj', 'HR', 40000);

INSERT INTO employees (EMP_ID, NAME, DEPT, SALARY) 
VALUES (104, 'Priya', 'HR', 45000);
~~~
### Example -- Ranking Employees by Salary

``` sql
SELECT
name,
dept,
salary,
RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS rank_in_dept
FROM employees;
```

### Real-time Scenario

A company wants to identify the **top salary employee in each
department**.

------------------------------------------------------------------------

### Example -- Running Total

``` sql
SELECT
name,
salary,
SUM(salary) OVER (ORDER BY salary) AS running_total
FROM employees;
```

**Use case** - Financial dashboards - Sales trend analysis

------------------------------------------------------------------------

# 2. Data Quality Operators in Oracle

Data quality operators help validate and clean data before analytics.
Oracle 23c and 23ai include enhancements to the Data Quality Operators for better data validation, standardization, and cleaning. These operators are critical in ensuring that the data used for analytics is accurate, consistent, and usable.

Key Enhancements:

Data Quality Functions: Oracle now includes more advanced functions for data profiling, data standardization, and data validation. These functions allow you to clean and transform raw data before it's used in analytics.

Fuzzy Matching: Enhanced fuzzy matching and duplicate detection operators help identify records that might represent the same entity but are written differently (e.g., different spellings of names, addresses).

Data Transformation: New operators for transforming data into the required format, such as converting text to proper case, handling missing data, and standardizing numerical values.

Using CLEAN_DATA Function
SELECT CLEAN_DATA(address) AS clean_address
FROM customer_data;

### Common Operators

  Operator      Purpose
  ------------- ------------------------
  IS NULL       Find missing data
  REGEXP_LIKE   Validate patterns
  TRIM          Remove extra spaces
  UPPER/LOWER   Standardize text
  CASE          Apply validation rules

### Example Table: CUSTOMERS

  ID   EMAIL           PHONE
  ---- --------------- ------------
  1    abc@gmail.com   9876543210
  2    abcgmail.com    NULL

### Example -- Find Missing Data

``` sql
SELECT *
FROM customers
WHERE phone IS NULL;
```

**Scenario** Identify customers without phone numbers.

------------------------------------------------------------------------

### Example -- Validate Email Format

``` sql
SELECT *
FROM customers
WHERE NOT REGEXP_LIKE(email,
'^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');
```

**Scenario** Detect invalid email addresses.

------------------------------------------------------------------------

### Example -- Clean Data

``` sql
SELECT
UPPER(TRIM(name)) AS clean_name
FROM employees;
```

**Scenario** Standardize names by removing spaces and converting to
uppercase.

------------------------------------------------------------------------

# 3. JSON Materialized View Enhancement

Oracle supports storing JSON data and optimizing queries using
materialized views.

A **Materialized View** stores the result of a query physically to
improve performance.

Materialized Views are a powerful feature in Oracle that allows you to store the results of complex queries for efficient querying. Oracle 23c and 23ai have introduced JSON Materialized Views, which allow you to store JSON data in materialized views and automatically refresh them as the underlying data changes.

Key Enhancements:

Storing JSON in Materialized Views: You can now create materialized views that store JSON data or JSON fragments extracted from relational data. This is useful when you need to perform analytics on the data without repeatedly parsing JSON documents.

Improved Performance: Enhanced indexing and caching of JSON data in materialized views significantly improves the performance of queries involving JSON documents.

Automatic Refresh: Materialized Views can be set to automatically refresh based on changes in the underlying data, making them ideal for real-time analytics.

This materialized view creates a JSON object for each row in the employees table and stores it in the view.

REFRESH COMPLETE ensures that the materialized view is refreshed whenever the base data changes.

### Create JSON Table

``` sql
CREATE TABLE orders_json (
id NUMBER,
order_data CLOB CHECK (order_data IS JSON)
);
```

Example JSON:

``` json
{
 "customer":"Amit",
 "amount":5000,
 "city":"Mumbai"
}
```

### Create JSON Materialized View

``` sql
CREATE MATERIALIZED VIEW orders_mv
BUILD IMMEDIATE
REFRESH FAST
AS
SELECT
id,
JSON_VALUE(order_data, '$.customer') AS customer,
JSON_VALUE(order_data, '$.amount') AS amount
FROM orders_json;
```

### Query the Materialized View

``` sql
SELECT * FROM orders_mv;
```

### Real-time Scenario

An e‑commerce application stores order details as JSON.\
Repeated JSON parsing becomes slow.

**Solution:** Create a materialized view to store extracted JSON values.

**Benefits** - Faster queries - Reduced CPU usage - Better reporting
performance

------------------------------------------------------------------------

# 4. Data Quality Constraints in Oracle

Oracle also supports constraints to ensure valid data.

### NOT NULL Constraint

``` sql
CREATE TABLE employees(
emp_id NUMBER,
name VARCHAR2(50) NOT NULL
);
```

Ensures name cannot be empty.

------------------------------------------------------------------------

### CHECK Constraint

``` sql
CREATE TABLE employees(
salary NUMBER,
CHECK (salary > 0)
);
```

Ensures salary is always positive.

------------------------------------------------------------------------

# Real-Time Data Quality Scenario

In a banking customer database:

Common problems: - Missing phone numbers - Invalid email formats -
Incorrect records

### Data Validation Query

``` sql
SELECT *
FROM customers
WHERE phone IS NULL
OR NOT REGEXP_LIKE(email,'@');
```

Used during **data cleaning pipelines before analytics**.

------------------------------------------------------------------------

# Summary

  Topic                     Purpose
  ------------------------- ----------------------------------------
  Analytic Functions        Advanced row-based analysis
  Data Quality Operators    Validate and clean data
  JSON Materialized Views   Speed up JSON queries
  Constraints               Ensure valid data during insert/update
