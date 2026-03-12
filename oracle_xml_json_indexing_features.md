# Oracle Database Advanced Indexing & XML Features

This document explains the following Oracle Database features with
**syntax, examples, and usage**:

1.  Transportable Binary XML (TBX)
2.  Indexes with Automatic Maintenance
3.  Enhanced Automatic Indexing
4.  JSON Search Index

------------------------------------------------------------------------

# 1. Transportable Binary XML (TBX)

## What is Transportable Binary XML?

Transportable Binary XML (TBX) is an **Oracle XML DB storage format**
used to store XML data in a **compact binary format** instead of plain
text.

### Advantages

-   Faster XML parsing
-   Reduced storage space
-   Efficient querying using XML indexes
-   Easy transport between databases

------------------------------------------------------------------------

## Syntax

``` sql
CREATE TABLE table_name OF XMLTYPE
XMLTYPE STORE AS BINARY XML;
```

------------------------------------------------------------------------

## Example

### Create Table with Binary XML Storage

``` sql
CREATE TABLE purchase_orders OF XMLTYPE
XMLTYPE STORE AS BINARY XML;
```

### Insert XML Data

``` sql
INSERT INTO purchase_orders VALUES (
XMLTYPE('<order>
            <id>101</id>
            <customer>John</customer>
            <amount>500</amount>
        </order>')
);
```

### Query XML Data

``` sql
SELECT x.object_value.extract('/order/customer/text()').getStringVal()
FROM purchase_orders x;
```

------------------------------------------------------------------------

## Usage

TBX is commonly used when:

-   Handling **large XML datasets**
-   Performing **XML queries frequently**
-   Storing **XML documents in enterprise applications**
-   Improving **performance of XML DB operations**

------------------------------------------------------------------------

# 2. Indexes with Automatic Maintenance

## What is Automatic Index Maintenance?

Automatic index maintenance means Oracle automatically **updates indexes
when table data changes**.

Whenever:

-   INSERT
-   UPDATE
-   DELETE

operations occur, the associated indexes are automatically updated.

------------------------------------------------------------------------

## Example

### Create Table

``` sql
CREATE TABLE employees (
    emp_id NUMBER,
    name VARCHAR2(100),
    salary NUMBER
);
```

### Create Index

``` sql
CREATE INDEX emp_salary_idx
ON employees(salary);
```

### Insert Data

``` sql
INSERT INTO employees VALUES (1,'John',5000);
INSERT INTO employees VALUES (2,'Alice',7000);
```

Oracle automatically updates the index **emp_salary_idx** when data
changes.

------------------------------------------------------------------------

## Usage

Automatic maintenance ensures:

-   Indexes stay **consistent with table data**
-   No manual index update required
-   Faster query performance

Example query benefiting from index:

``` sql
SELECT * FROM employees
WHERE salary = 7000;
```

------------------------------------------------------------------------

# 3. Enhanced Automatic Indexing

## What is Automatic Indexing?

Automatic Indexing is a feature where **Oracle automatically creates,
tests, and manages indexes** to improve SQL performance.

Oracle:

1.  Monitors SQL workload
2.  Creates candidate indexes
3.  Tests them invisibly
4.  Makes them visible if performance improves

------------------------------------------------------------------------

## Enable Automatic Indexing

``` sql
EXEC DBMS_AUTO_INDEX.CONFIGURE('AUTO_INDEX_MODE','IMPLEMENT');
```

Modes:

| Mode \| Description \|

\|-----\|-------------\| REPORT ONLY \| Monitor recommendations \| \|
IMPLEMENT \| Automatically create indexes \| \| OFF \| Disable feature
\|

------------------------------------------------------------------------

## Check Automatic Indexes

``` sql
SELECT index_name, table_name, visibility
FROM dba_indexes
WHERE auto = 'YES';
```

------------------------------------------------------------------------

## Example

If frequent query:

``` sql
SELECT * FROM orders
WHERE customer_id = 100;
```

Oracle may automatically create:

``` sql
CREATE INDEX SYS_AI_ORDERS_CUST
ON orders(customer_id);
```

------------------------------------------------------------------------

## Usage

Best used for:

-   Performance tuning
-   Large enterprise databases
-   Systems with unpredictable query patterns

Benefits:

-   Reduces manual tuning
-   Improves SQL performance automatically

------------------------------------------------------------------------

# 4. JSON Search Index

## What is JSON Search Index?

A JSON Search Index allows **fast searching of JSON documents stored in
Oracle Database**.

It is based on **Oracle Text technology**.

------------------------------------------------------------------------

## Syntax

``` sql
CREATE SEARCH INDEX index_name
ON table_name(column_name)
FOR JSON;
```

------------------------------------------------------------------------

## Example

### Create Table

``` sql
CREATE TABLE products (
    id NUMBER,
    product_info JSON
);
```

### Insert JSON Data

``` sql
INSERT INTO products VALUES(
1,
'{
 "name":"Laptop",
 "brand":"Dell",
 "price":800
}'
);
```

------------------------------------------------------------------------

### Create JSON Search Index

``` sql
CREATE SEARCH INDEX product_json_idx
ON products(product_info)
FOR JSON;
```

------------------------------------------------------------------------

### Query JSON Data

``` sql
SELECT *
FROM products
WHERE JSON_EXISTS(product_info,'$.brand?(@ == "Dell")');
```

------------------------------------------------------------------------

## Usage

JSON Search Index is used when:

-   Working with **JSON documents**
-   Building **NoSQL-style applications**
-   Fast searching inside JSON fields

Benefits:

-   Fast JSON queries
-   Full text search
-   Structured JSON filtering

------------------------------------------------------------------------

# Summary

| Feature \| Purpose \|

\|------\|------\| Transportable Binary XML \| Efficient XML storage \|
\| Automatic Index Maintenance \| Keeps indexes synchronized \| \|
Enhanced Automatic Indexing \| Automatically creates indexes \| \| JSON
Search Index \| Fast searching of JSON data \|

------------------------------------------------------------------------

# Conclusion

These Oracle database features help improve:

-   **Performance**
-   **Data storage efficiency**
-   **Automatic optimization**
-   **Search capabilities for XML and JSON**

They are widely used in **modern enterprise Oracle database systems**.
