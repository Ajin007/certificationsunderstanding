
# Oracle 23c / 23ai New Features (JSON & Data Model Enhancements)

## 1. JSON Schema Support

### Description
Oracle supports JSON Schema validation directly in the database to enforce structure and rules for JSON documents.

### Syntax
```sql
JSON_SCHEMA_VALID(schema, json_document)
```

### Example
```sql
CREATE TABLE employees_json (
    data JSON
);
```

Register Schema

```sql
**********Schema available in the database or not to check the same use it ******************
SELECT object_name, object_type
FROM all_objects
WHERE object_name = 'DBMS_JSON_SCHEMA';
******************************************************************************************************

DECLARE
schema_doc JSON := '{
  "type": "object",
  "required": ["id","name"],
  "properties": {
     "id": {"type":"number"},
     "name": {"type":"string"},
     "salary": {"type":"number"}
  }
}';
BEGIN
  DBMS_JSON_SCHEMA.register_schema(
      schema_name => 'emp_schema',
      schema_doc  => schema_doc
  );
END;
/
```

Insert JSON

```sql
INSERT INTO employees_json VALUES (
JSON('{"id":101,"name":"Rahul","salary":5000}')
);
```

### Usage
- Enforcing JSON structure
- API data validation
- Data integrity

---

## 2. SODA Enhancement

### Description
SODA (Simple Oracle Document Access) allows JSON documents to be managed like NoSQL collections.

### Create Collection

```sql
***************************
## check for the soda 
SELECT * FROM all_objects WHERE object_name = 'DBMS_SODA';
******************************
BEGIN
  DBMS_SODA.create_collection(
     collection_name => 'customers'
  );
END;
/
```

### Insert Document

```sql
DECLARE
  doc SODA_DOCUMENT_T;
BEGIN
  doc := SODA_DOCUMENT_T(
    b_content => '{"id":1,"name":"Amit","city":"Delhi"}'
  );

  DBMS_SODA.insert_one(
     collection_name => 'customers',
     document => doc
  );
END;
/
```

### Usage
- NoSQL applications
- Document-based storage
- Semi-structured data

---

## 3. JSON_TRANSFORM Enhancement

### Description
Used to modify JSON documents in-place.

### Syntax
```sql
JSON_TRANSFORM(
   json_column,
   operation path = value
)
```

### Example

```sql
CREATE TABLE products (
  info JSON
);
```

Insert JSON

```sql
INSERT INTO products VALUES(
JSON('{"id":1,"name":"Laptop","price":50000}')
);
```

Update JSON Field

```sql
UPDATE products
SET info =
JSON_TRANSFORM(
   info,
   SET '$.price' = 55000
);
```

Add Attribute

```sql
UPDATE products
SET info =
JSON_TRANSFORM(
   info,
   INSERT '$.brand' = 'Dell'
);
```

### Usage
- Partial JSON updates
- Better performance
- Avoid full document replacement

---

## 4. Comparing and Sorting JSON Data Type

### Description
Oracle 23c allows direct comparison and sorting of JSON datatype.

### Example

```sql
CREATE TABLE orders (
  details JSON
);
```

Insert Data

```sql
INSERT INTO orders VALUES (JSON('{"id":1,"amount":200}'));
INSERT INTO orders VALUES (JSON('{"id":2,"amount":150}'));
```

Sorting

```sql
SELECT *
FROM orders
ORDER BY details;
```

### Usage
- Native JSON operations
- Sorting JSON documents

---

## 5. Predicates for JSON_VALUE and JSON_QUERY

### Description
Predicates allow conditions inside JSON path expressions.

### Syntax
```sql
JSON_VALUE(json_column, '$?(@.attribute condition)')
```

### Example

```sql
CREATE TABLE employees (
   data JSON
);
```

Insert JSON

```sql
INSERT INTO employees VALUES(
JSON('{"id":1,"name":"Ravi","salary":7000}')
);
```

Query Using Predicate

```sql
SELECT *
FROM employees
WHERE JSON_VALUE(data, '$?(@.salary > 6000).name') IS NOT NULL;
```

### Usage
- Filtering JSON data
- Querying nested attributes

---

## 6. Relational Duality Views

### Description
Relational Duality Views allow relational data to be represented as JSON documents.

### Example Tables

```sql
CREATE TABLE customers (
  id NUMBER PRIMARY KEY,
  name VARCHAR2(100)
);

CREATE TABLE orders (
  order_id NUMBER,
  customer_id NUMBER,
  amount NUMBER
);
```

### Create Duality View

```sql
CREATE OR REPLACE JSON RELATIONAL DUALITY VIEW customer_orders
AS
SELECT JSON {
   'id' : c.id,
   'name' : c.name,
   'orders' :
       (SELECT JSON_ARRAYAGG(
           JSON {
             'order_id' : o.order_id,
             'amount' : o.amount
           }
       )
       FROM orders o
       WHERE o.customer_id = c.id)
}
FROM customers c;
```

### Query

```sql
SELECT *
FROM customer_orders;
```

### Usage
- Modern application development
- JSON APIs
- Microservices

---

## Summary

| Feature | Purpose |
|------|------|
| JSON Schema | Validate JSON structure |
| SODA | Document-based NoSQL access |
| JSON_TRANSFORM | Modify JSON data |
| JSON Comparison | Compare/sort JSON data |
| JSON Predicates | Filter JSON data |
| Relational Duality View | Combine relational and JSON models |
