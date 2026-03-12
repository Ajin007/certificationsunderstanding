# Oracle Materialized Views -- Syntax, Examples, and Usage Guide

## 1. What is a Materialized View (MV)?

A **Materialized View** in Oracle is a database object that stores the
**result of a query physically**.\
Unlike a normal view, which runs the query every time it is accessed, a
materialized view **stores the data** and can be refreshed periodically.

### Advantages

-   Improves query performance
-   Pre-computed joins and aggregations
-   Useful for **data warehouses and reporting**
-   Reduces load on base tables

------------------------------------------------------------------------

# 2. Semi Join in Materialized View

## What is a Semi Join?

A **Semi Join** returns rows from the first table **only when a match
exists in the second table**.

Commonly implemented using:

-   `EXISTS`
-   `IN`

## Example Tables

``` sql
CUSTOMERS
---------
CUSTOMER_ID
CUSTOMER_NAME

ORDERS
------
ORDER_ID
CUSTOMER_ID
ORDER_DATE
```

## Semi Join using EXISTS

``` sql
CREATE MATERIALIZED VIEW mv_customer_orders
BUILD IMMEDIATE
REFRESH FAST ON COMMIT
AS
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

### Explanation

-   `EXISTS` creates a **semi join condition**
-   Only customers who have orders will appear
-   MV stores the result physically

------------------------------------------------------------------------

# 3. Materialized View using ANSI JOIN

ANSI joins use explicit join syntax like:

-   `INNER JOIN`
-   `LEFT JOIN`
-   `RIGHT JOIN`
-   `FULL JOIN`

## Syntax

``` sql
CREATE MATERIALIZED VIEW mv_sales_data
BUILD IMMEDIATE
REFRESH COMPLETE
ON DEMAND
AS
SELECT columns
FROM table1
INNER JOIN table2
ON condition;
```

## Example

Tables:

``` sql
EMPLOYEES
EMP_ID
EMP_NAME
DEPT_ID

DEPARTMENTS
DEPT_ID
DEPT_NAME
```

### Materialized View using ANSI JOIN

``` sql
CREATE MATERIALIZED VIEW mv_employee_dept
BUILD IMMEDIATE
REFRESH COMPLETE
ON DEMAND
AS
SELECT
    e.emp_id,
    e.emp_name,
    d.dept_name
FROM employees e
INNER JOIN departments d
ON e.dept_id = d.dept_id;
```

### Explanation

-   Uses **ANSI join syntax**
-   MV stores **joined data**
-   Useful for **reporting queries**

------------------------------------------------------------------------

# 4. Materialized View Refresh Methods

Materialized views can be refreshed in multiple ways.

## 1. COMPLETE REFRESH

Rebuilds the entire MV.

``` sql
REFRESH COMPLETE
```

Example:

``` sql
CREATE MATERIALIZED VIEW mv_sales
REFRESH COMPLETE
ON DEMAND
AS
SELECT * FROM sales;
```

### Characteristics

-   Deletes existing rows
-   Re-runs the full query
-   Slower for large tables

------------------------------------------------------------------------

## 2. FAST REFRESH

Only changed rows are updated.

Requirements:

-   Materialized View Logs must exist

Example:

``` sql
CREATE MATERIALIZED VIEW LOG ON sales
WITH ROWID, PRIMARY KEY
INCLUDING NEW VALUES;
```

Materialized View:

``` sql
CREATE MATERIALIZED VIEW mv_sales
REFRESH FAST
ON COMMIT
AS
SELECT * FROM sales;
```

### Characteristics

-   Very fast
-   Uses **change tracking**
-   Requires **MV logs**

------------------------------------------------------------------------

## 3. FORCE REFRESH

Oracle chooses between FAST or COMPLETE.

``` sql
REFRESH FORCE
```

Example:

``` sql
CREATE MATERIALIZED VIEW mv_sales
REFRESH FORCE
ON DEMAND
AS
SELECT * FROM sales;
```

------------------------------------------------------------------------

# 5. Concurrent Materialized View Refresh

Concurrent refresh allows **multiple sessions to read the MV while
refresh is happening**.

Introduced to reduce locking and improve availability.

### Example

``` sql
BEGIN
DBMS_MVIEW.REFRESH(
    list => 'MV_SALES',
    method => 'C',
    atomic_refresh => FALSE
);
END;
/
```

### Benefits

-   Reduces locking
-   Improves system availability
-   Better for large data warehouses

------------------------------------------------------------------------

# 6. DBMS_MVIEW Package (API)

Oracle provides **DBMS_MVIEW** package to manage materialized views.

Common procedures:

  Procedure           Purpose
  ------------------- -----------------------------------
  REFRESH             Refresh MV
  EXPLAIN_MVIEW       Check if MV supports fast refresh
  PURGE_LOG           Clean MV logs
  REFRESH_DEPENDENT   Refresh dependent MVs

------------------------------------------------------------------------

## Example: Refresh Materialized View

``` sql
BEGIN
DBMS_MVIEW.REFRESH('MV_EMPLOYEE_DEPT');
END;
/
```

------------------------------------------------------------------------

## Refresh Multiple MVs

``` sql
BEGIN
DBMS_MVIEW.REFRESH(
    list => 'MV1,MV2,MV3',
    method => 'C'
);
END;
/
```

------------------------------------------------------------------------

# 7. Refresh Enhancement Techniques

To improve MV refresh performance:

### 1. Partition Change Tracking (PCT)

Refresh only changed partitions.

### 2. Parallel Refresh

``` sql
ALTER SESSION ENABLE PARALLEL DML;
```

``` sql
BEGIN
DBMS_MVIEW.REFRESH(
    list => 'MV_SALES',
    method => 'C',
    parallelism => 4
);
END;
/
```

### 3. Atomic Refresh Control

``` sql
atomic_refresh => FALSE
```

Benefits:

-   Uses **TRUNCATE + INSERT**
-   Much faster refresh

------------------------------------------------------------------------

# 8. Concurrent Materialized View Example

``` sql
BEGIN
DBMS_MVIEW.REFRESH(
    list => 'MV_EMPLOYEE_DEPT',
    method => 'F',
    atomic_refresh => FALSE
);
END;
/
```

### Explanation

-   `method => 'F'` = FAST refresh
-   `atomic_refresh => FALSE` enables **concurrent refresh behavior**

------------------------------------------------------------------------

# 9. Best Practices

### Use FAST refresh whenever possible

Requires:

-   Primary key
-   MV logs

### Use COMPLETE refresh when

-   Complex joins
-   Aggregations
-   No MV logs

### Use ON COMMIT when

-   Real-time reporting needed

### Use ON DEMAND when

-   Batch refresh is acceptable

------------------------------------------------------------------------

# 10. Summary

Materialized Views provide:

-   Precomputed query results
-   Performance optimization
-   Efficient reporting

Important concepts:

  Feature              Purpose
  -------------------- ----------------------------
  Semi Join            Filter rows based on match
  ANSI Join            Modern join syntax
  FAST Refresh         Incremental updates
  COMPLETE Refresh     Full rebuild
  Concurrent Refresh   Reduce locking
  DBMS_MVIEW           API to manage MVs

------------------------------------------------------------------------

**End of Guide**
