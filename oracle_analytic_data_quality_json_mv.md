# Oracle Concepts: Analytic Platform, Data Quality Operators, JSON Materialized View

## 1. Oracle Analytic Functions (Analytic Platform)

Analytic functions in Oracle Database perform calculations across a set
of rows related to the current row while still returning individual
rows.

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
