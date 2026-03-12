# Oracle Database 23c / 23ai -- Advanced Query Processing & Materialized View Enhancements

Author: Generated for Study/Reference\
Topics Covered: - Vectorized Query Processing - LPCT (Logical Partition
Change Tracking) - Staging Tables for Materialized View Refresh -
Multi‑level Joins and Aggregation Views

------------------------------------------------------------------------

# 1. Vectorized Query Processing

## What it is

Vectorized Query Processing is an optimization feature introduced in
Oracle 23c that processes rows in **batches (vectors)** instead of **one
row at a time**.

Traditional execution:

    Row → Process → Row → Process → Row → Process

Vectorized execution:

    Batch of Rows → Process Together → Batch of Rows

This significantly improves performance for:

-   Analytical queries
-   Aggregations
-   Joins
-   Data warehouse workloads

It reduces:

-   CPU overhead
-   Function call overhead
-   Memory access cost

------------------------------------------------------------------------

## Syntax

Vectorized processing is usually enabled automatically by the optimizer.

But you can influence execution using hints.

    SELECT /*+ VECTOR_TRANSFORM */
           column_list
    FROM   table_name;

or

    SELECT /*+ VECTOR_GROUP_BY */
           column_list
    FROM   table_name
    GROUP BY columns;

------------------------------------------------------------------------

## Example

### Create Table

``` sql
CREATE TABLE sales (
    sale_id NUMBER,
    product_id NUMBER,
    amount NUMBER,
    sale_date DATE
);
```

### Query Using Vector Group By

``` sql
SELECT /*+ VECTOR_GROUP_BY */
       product_id,
       SUM(amount) total_sales
FROM   sales
GROUP BY product_id;
```

### Execution Plan

Vectorized operators appear like:

    VECTOR GROUP BY
    VECTOR JOIN
    VECTOR TRANSFORM

------------------------------------------------------------------------

## Usage Scenarios

Best for:

-   Data warehouse queries
-   Star schema joins
-   Large aggregations
-   Analytics workloads

------------------------------------------------------------------------

# 2. LPCT -- Logical Partition Change Tracking

## What it is

LPCT stands for:

**Logical Partition Change Tracking**

It improves **Materialized View refresh performance**.

Traditional refresh methods:

  Method     Description
  ---------- ---------------------------
  COMPLETE   Full rebuild
  FAST       Uses logs
  PCT        Partition change tracking

LPCT allows Oracle to track **logical partitions instead of physical
ones**.

Benefits:

-   Faster refresh
-   No strict partition alignment needed
-   Less dependency on partitioned base tables

------------------------------------------------------------------------

## Syntax

Enable LPCT when creating a Materialized View.

``` sql
CREATE MATERIALIZED VIEW sales_mv
REFRESH FAST
ENABLE LOGICAL PARTITION CHANGE TRACKING
AS
SELECT product_id,
       SUM(amount) total_sales
FROM sales
GROUP BY product_id;
```

------------------------------------------------------------------------

## Example

### Base Table

``` sql
CREATE TABLE orders (
    order_id NUMBER,
    customer_id NUMBER,
    order_date DATE,
    amount NUMBER
);
```

### Create Materialized View

``` sql
CREATE MATERIALIZED VIEW orders_summary_mv
BUILD IMMEDIATE
REFRESH FAST
ENABLE LOGICAL PARTITION CHANGE TRACKING
AS
SELECT customer_id,
       SUM(amount) total_amount
FROM orders
GROUP BY customer_id;
```

### Refresh

``` sql
EXEC DBMS_MVIEW.REFRESH('ORDERS_SUMMARY_MV','FAST');
```

Oracle refreshes **only logically changed data**.

------------------------------------------------------------------------

# 3. Staging Tables for Materialized View Refresh

## What it is

Oracle 23c introduces **staging tables** to improve materialized view
refresh performance.

A **staging table** temporarily holds data before applying it to the MV.

Advantages:

-   Parallel refresh
-   Reduced locking
-   Better refresh performance
-   Scalable ETL pipelines

------------------------------------------------------------------------

## Syntax

``` sql
CREATE MATERIALIZED VIEW sales_mv
REFRESH FAST
USING STAGING TABLE
AS
SELECT product_id,
       SUM(amount) total_sales
FROM sales
GROUP BY product_id;
```

------------------------------------------------------------------------

## Example

``` sql
CREATE MATERIALIZED VIEW sales_summary_mv
BUILD IMMEDIATE
REFRESH FAST
USING STAGING TABLE
AS
SELECT product_id,
       SUM(amount) total_sales
FROM sales
GROUP BY product_id;
```

During refresh:

1.  Changes loaded into staging table
2.  Oracle merges changes into MV

------------------------------------------------------------------------

# 4. Staleness Tracking for Materialized Views

Oracle tracks whether a Materialized View is **fresh or stale**.

Possible states:

| Status \| Meaning \|

\|------\|------\| FRESH \| Up‑to‑date \| \| STALE \| Base table changed
\| \| NEEDS_COMPILE \| Requires recompilation \| \| UNUSABLE \| Cannot
be used \|

------------------------------------------------------------------------

## Check Staleness

``` sql
SELECT mview_name,
       staleness
FROM   user_mviews;
```

Example output:

    MVIEW_NAME           STALENESS
    ------------------   ---------
    SALES_SUMMARY_MV     STALE

------------------------------------------------------------------------

# 5. Multi‑Level Joins and Aggregation Views

Oracle 23c enhances query optimization for complex analytical queries
involving:

-   Multiple joins
-   Nested aggregations
-   Star schema queries

These are common in **data warehouse systems**.

------------------------------------------------------------------------

## Example Schema

Tables:

-   customers
-   orders
-   products
-   sales

------------------------------------------------------------------------

## Multi-Level Join Example

``` sql
SELECT c.customer_id,
       p.product_id,
       SUM(s.amount) total_sales
FROM customers c
JOIN orders o
     ON c.customer_id = o.customer_id
JOIN sales s
     ON o.order_id = s.order_id
JOIN products p
     ON s.product_id = p.product_id
GROUP BY c.customer_id,
         p.product_id;
```

Oracle optimizer can apply:

-   Vector joins
-   Join elimination
-   Aggregation pushdown

------------------------------------------------------------------------

# Aggregation View Example

Aggregation views simplify complex analytics.

### Create Aggregation View

``` sql
CREATE OR REPLACE VIEW product_sales_summary AS
SELECT product_id,
       SUM(amount) total_sales,
       COUNT(*) total_orders
FROM sales
GROUP BY product_id;
```

### Query

``` sql
SELECT *
FROM product_sales_summary
WHERE total_sales > 10000;
```

------------------------------------------------------------------------

# Performance Benefits in Oracle 23c / 23ai

  Feature                 Benefit
  ----------------------- -------------------------------
  Vectorized Processing   Faster analytics
  LPCT                    Faster MV refresh
  Staging Tables          Efficient incremental loads
  Staleness Tracking      Query rewrite safety
  Multi‑Level Joins       Optimized star schema queries

------------------------------------------------------------------------

# Real‑World Usage

These features are mainly used in:

-   Data Warehousing
-   Real‑time analytics
-   ETL pipelines
-   Reporting systems
-   Large aggregation queries

Industries:

-   Banking analytics
-   E‑commerce reporting
-   Telecom data processing
-   Financial dashboards

------------------------------------------------------------------------

# Summary

Oracle 23c / 23ai introduces powerful query processing and materialized
view improvements.

Key capabilities:

-   Vectorized query execution
-   Logical partition change tracking
-   Staging tables for MV refresh
-   Staleness monitoring
-   Efficient multi‑level joins and aggregations

These significantly improve performance for **large scale analytical
workloads**.

------------------------------------------------------------------------
