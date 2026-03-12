# Netflix recommended system
## creation of the table
## CONCEPTS:
[1] what is vector embedding ?
<img width="1739" height="734" alt="image" src="https://github.com/user-attachments/assets/429854ee-9dfe-4e35-86c5-c684754174cc" />
[2] Vector Embedding models
<img width="1741" height="764" alt="image" src="https://github.com/user-attachments/assets/882211b4-823e-4925-9f4f-b83d0e07a61a" />
<img width="1371" height="610" alt="image" src="https://github.com/user-attachments/assets/1e5518e6-04af-4f3d-ab31-24d795e76934" />
| Data Type                   | Example Embedding Models                    | Input Data                      | Output (Vector)                              | Typical Dimensions | Use Cases                                                   | How It Is Used in Oracle DB                                                                         |
| --------------------------- | ------------------------------------------- | ------------------------------- | -------------------------------------------- | ------------------ | ----------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| **Text Embedding**          | text-embedding-3-large, BERT, Sentence-BERT | Sentences, documents, queries   | Numeric vector representing semantic meaning | 768, 1024, 1536    | Semantic search, RAG systems, chatbots, document similarity | Text converted to embeddings → stored in VECTOR column → similarity search with `VECTOR_DISTANCE()` |
| **Image Embedding**         | CLIP, ResNet                                | Images (photos, product images) | Vector representing visual features          | 512, 768           | Image search, facial recognition, product similarity        | Image converted to embedding → stored in VECTOR column → search for visually similar images         |
| **Voice / Audio Embedding** | Whisper, Wav2Vec                            | Speech recordings, audio files  | Vector representing speech features          | 512, 768           | Voice search, call center analytics, speaker recognition    | Audio converted to vector → stored in Oracle → similarity search on voice patterns                  |


## Comparison of vector not allowed in querry:
<img width="1742" height="860" alt="image" src="https://github.com/user-attachments/assets/a7f39b07-bdad-4311-8427-e882a043babd" />


### AI models convert text (movie description, product description, etc.) into embeddings (vectors).
```
CREATE TABLE netflix_movies (
    movie_id NUMBER,
    movie_title VARCHAR2(100),
    genre VARCHAR2(50),
    embedding VECTOR(4)
);
```
### Syntax of the Vector
```
VECTOR(<dimension>)
## ---------------------------------
For the VECTOR(n) datatype:

Minimum dimension: 1

Maximum dimension: 65535

## ------------------------------
VECTOR(4,*,DENSE)
         ↑
    FLOAT32
     FLOAT64
    INT8
| Part                        | Meaning                          |
| --------------------------- | -------------------------------- |
| `EMBEDDED_FROM_FILE_IMAGES` | Column name                      |
| `VECTOR`                    | Vector datatype                  |
| `4`                         | Vector has **4 dimensions**      |
| `*`                         | **Any numeric format** allowed   |
| `DENSE`                     | **All values stored explicitly** |


## --------------------------------
Works with functions like:

VECTOR_DISTANCE

VECTOR_DOT_PRODUCT

VECTOR_COSINE_DISTANCE
```
## Addition of the table
```
INSERT INTO netflix_movies VALUES
(1,'Avengers','Action', VECTOR('[0.9,0.8,0.7,0.6]'));

INSERT INTO netflix_movies VALUES
(2,'Batman','Action', VECTOR('[0.85,0.75,0.72,0.65]'));

INSERT INTO netflix_movies VALUES
(3,'Titanic','Romance', VECTOR('[0.2,0.1,0.3,0.4]'));

INSERT INTO netflix_movies VALUES
(4,'Notebook','Romance', VECTOR('[0.25,0.15,0.35,0.45]'));

INSERT INTO netflix_movies VALUES
(5,'Interstellar','SciFi', VECTOR('[0.6,0.7,0.8,0.9]'));

COMMIT;
```

## Syntax of the Vector_Distance
~~~
VECTOR_DISTANCE(vector_column, query_vector, distance_metric)

| Parameter         | Meaning                                 |
| ----------------- | --------------------------------------- |
| `vector_column`   | Column that stores vectors in the table |
| `query_vector`    | Vector you want to compare with         |
| `distance_metric` | Method used to calculate similarity     |

Example: VECTOR_DISTANCE(embedding, VECTOR(0.9,0.8,0.7,0.6), COSINE)

*** Cosine can be replaced by 
| Metric      | Meaning                            |
| ----------- | ---------------------------------- |
| `COSINE`    | Angle similarity (most used in AI) |
| `EUCLIDEAN` | Straight-line distance             |
| `DOT`       | Dot product similarity             |
NOTR: Hamming similarity, Manhattan Distance , Euclidean square distance matrix
    ----> cosine
    Meaning:
    Measures the angle between two vectors, not their magnitude.
    If the angle is small → vectors are very similar.

    ----> Euclidean Distance
    Meaning:
    Measures the straight-line distance between two vectors in space.
~~~
<img width="1134" height="406" alt="image" src="https://github.com/user-attachments/assets/b7898ba3-4db4-465e-a5f9-39a3b7316263" />

~~~
    ******  Vectors:*****
    A = [1,2]
    B = [4,6]
    ****** Distance ******
    sqrt((1-4)^2 + (2-6)^2)
    = sqrt(9 + 16)
    = 5
    **** Interpretation ****
            | Distance | Meaning     |
        | -------- | ----------- |
        | Small    | Similar     |
        | Large    | Not similar |
    *** used in ***
    image similarity
    clustering
    recommendation systems

    ---> Dot product
    | Value          | Meaning   |
| -------------- | --------- |
| Large positive | Similar   |
| 0              | unrelated |
| Negative       | opposite  |
 Used heavily in:
neural networks
transformers
attention mechanisms


~~~
## Example of the Vector_distance:
## How to get the recommended films alone
~~~
select movie_title,
VECTOR_DISTANCE(embedded_from_file_images,vector('[0.9,0.8,0.7,0.6]'),cosine) as similarity
from NETFLIX_MOVIES
order by similarity
~~~

## How to fetch the first three in the recommended list
~~~
select movie_title,
VECTOR_DISTANCE(embedded_from_file_images,vector('[0.9,0.8,0.7,0.6]'),cosine) as similarity
from NETFLIX_MOVIES
order by similarity
fetch first 3 rows only;
~~~

## Attribute filtering
~~~
SELECT movie_title
FROM netflix_movies
WHERE genre = 'Action'
ORDER BY VECTOR_DISTANCE(embedding, VECTOR(0.9,0.8,0.7,0.6), COSINE);
~~~
# create a Product table and do the query for the same:
~~~
CREATE TABLE amazon_products (
    product_id NUMBER,
    product_name VARCHAR2(100),
    category VARCHAR2(50),
    embedding VECTOR(4)
);
INSERT INTO amazon_products VALUES
(1,'Wireless Gaming Mouse','Electronics', VECTOR(0.88,0.75,0.66,0.59));

INSERT INTO amazon_products VALUES
(2,'Gaming Keyboard','Electronics', VECTOR(0.82,0.70,0.60,0.55));

INSERT INTO amazon_products VALUES
(3,'Bluetooth Headphones','Electronics', VECTOR(0.70,0.60,0.50,0.40));

INSERT INTO amazon_products VALUES
(4,'Office Chair','Furniture', VECTOR(0.20,0.25,0.30,0.35));

INSERT INTO amazon_products VALUES
(5,'Laptop Stand','Accessories', VECTOR(0.40,0.45,0.50,0.55));

COMMIT;

###-------- how search millions of data easily
CREATE VECTOR INDEX amazon_vector_idx
ON amazon_products(embedding);

~~~
## how to do the search effectively 
~~~
###----------------------------------------------------------------
In AI Vector Search (like in Oracle Database 23ai), special indexing techniques are used to quickly find similar vectors among millions of records. Two important ones are:

HNSW (Hierarchical Navigable Small World Index)

IVF (Inverted File Index)

Both help speed up similarity search, such as recommending movies in Netflix.

#### -----------------------------------------
1️⃣ HNSW (Hierarchical Navigable Small World Index)
Concept
HNSW creates a multi-layer graph structure where each vector is connected to its nearest neighbors.

Instead of checking every vector, the algorithm navigates through neighbors to find the closest ones.

Think of it like a social network of similar movies.
********************Syntax********************
CREATE VECTOR INDEX index_name
ON table_name(vector_column)
ORGANIZATION HNSW
DISTANCE COSINE;
### -------------------
2️⃣ IVF (Inverted File Index)
Concept

IVF divides vectors into clusters (groups).

Instead of searching the entire dataset, it searches only the closest clusters.

Think of it like movie genres or categories.
*************Syntax **********
CREATE VECTOR INDEX index_name
ON table_name(vector_column)
ORGANIZATION IVF
DISTANCE COSINE;

~~~
## how to select the oracle version
~~~
SELECT banner FROM v$version;
~~~
## how to add a column with vector
~~~
ALTER TABLE netflix_movies
ADD (embedded_new VECTOR(4));
~~~
## How to copy data from the old column
~~~
UPDATE netflix_movies
SET embedded_new = embedded_from_file_images;
~~~
## Drop the old vector column
~~~
ALTER TABLE netflix_movies
DROP COLUMN embedded_from_file_images;
~~~
## How to check all the parameters 
~~~
SELECT parameter, value FROM v$option
ORDER BY parameter;
~~~
## Default on Null statement
~~~
In Oracle Database 23ai (and earlier versions), the clause DEFAULT ON NULL is used only for INSERT operations, not for UPDATE statements.
Example:
CREATE TABLE employeesNull (
    emp_id NUMBER,
    salary NUMBER DEFAULT ON NULL 5000
)
insert INTO EMPLOYEESNULL values(3,null)
~~~
## Annotation tags
~~~
Annotation is simply a KEY = VALUE attached to the database object.
AI datacatalog , metadata, Used by the AI Tools, easily searchable


[1]. Method level Annotation
create table employeeAnnotation(
    emp_id NUMBER
    ANNOTATIONS(
        identity 'primary'
    )
    ,  
    emp_dept VARCHAR2(57)
    ANNOTATIONS(
        pririty 'high',
        sensitivity 'medium'
    )
)

[2]. Table level annotations:
-- CREATE TABLE employeeAnnotationTable (
--     emp_id NUMBER,
--     emp_dept VARCHAR2(50)
-- )
-- ANNOTATIONS (
--     owner 'HR Department',
--     classification 'internal',
--     sensitivity 'medium'
-- );

SELECT OBJECT_NAME,
       COLUMN_NAME,
       ANNOTATION_NAME,
       ANNOTATION_VALUE
FROM USER_ANNOTATIONS_USAGE
WHERE OBJECT_NAME = 'EMPLOYEEANNOTATIONTABLE';
-- column_name null means it is table level annotation

~~~
## If not exists clause update
~~~
create table if not exists  demo (
    id NUMBER,
    demo_name VARCHAR2(244)
)
-- drop table if exists demo;
~~~
## group by position using the session set 
~~~

ALTER SESSION SET GROUP_BY_POSITION_ENABLED=TRUE;
-- position of genre in the slect list 
select genre,count(*) as movie_count from netflix_movies group by 1
~~~

## select without form 

~~~
-- no need to use the dual class
1. select sysdate from dual;
select sysdate
~~~

## sql update return clause enhancement with bind varaible 
~~~
-- Drop and recreate table for demo
DROP TABLE employees PURGE;

CREATE TABLE employees (
    employee_id NUMBER PRIMARY KEY,
    name        VARCHAR2(100),
    salary      NUMBER
);

-- Insert sample data
INSERT INTO employees (employee_id, name, salary)
VALUES (100, 'John Doe', 5000);

INSERT INTO employees (employee_id, name, salary)
VALUES (101, 'Alice Smith', 6000);

COMMIT;

-- -------------------------------
-- Declare bind variables
-- -------------------------------
VARIABLE l_id      NUMBER
VARIABLE l_old_sal NUMBER

-- Bind the value of 100 to the l_id variable
EXEC :l_id := 100;

-- -------------------------------
-- Update using RETURNING clause
-- -------------------------------
-- Update the salary for employee 100 and capture old salary
UPDATE employees
SET salary = salary * 2
WHERE employee_id = :l_id
RETURNING salary/2 INTO :l_old_sal;

-- -------------------------------
-- View bind variable values
-- -------------------------------
PRINT l_id       -- should print 100
PRINT l_old_sal  -- should print the old salary (5000)

-- -------------------------------
-- Verify updated salary
-- -------------------------------
SELECT employee_id, salary
FROM employees
WHERE employee_id = :l_id;

-- -------------------------------
-- Rollback if needed
-- -------------------------------
ROLLBACK;

-- -------------------------------
-- Verify rollback
-- -------------------------------
SELECT employee_id, salary
FROM employees
WHERE employee_id = :l_id;
~~~


## Boolean return type 
~~~
CREATE TABLE employee_status (
  emp_id NUMBER PRIMARY KEY,
  is_active BOOLEAN
);

INSERT INTO employee_status VALUES (1001, TRUE);
INSERT INTO employee_status VALUES (1002, yes);
INSERT INTO employee_status VALUES (1003, n);

SELECT * 
FROM employee_status
WHERE is_active;

~~~

## Value Constrcutor
~~~
drop table employee_status;
CREATE TABLE employee_status (
    emp_id NUMBER PRIMARY KEY,
    emp_name VARCHAR2(50),
    is_active BOOLEAN,
    department VARCHAR2(30)
);

***************Insert multiple rows*********
INSERT INTO employee_status (emp_id, emp_name, is_active, department)
VALUES 
    (101, 'Alice', TRUE, 'HR'),
    (102, 'Bob', FALSE, 'DEV'),
    (103, 'Charlie', TRUE, 'AI'),
    (104, 'Diana', NULL, 'OPS');

************* 
~~~
## Aggregate interval
~~~
🔹 1. What is an INTERVAL in Oracle?

Oracle supports two main interval types:

INTERVAL YEAR TO MONTH – Represents a number of years and months.
Example: INTERVAL '2-6' YEAR TO MONTH → 2 years 6 months.

INTERVAL DAY TO SECOND – Represents days, hours, minutes, seconds.
Example: INTERVAL '3 04:30:15.123' DAY TO SECOND → 3 days, 4 hours, 30 minutes, 15.123 seconds.

These types are used to store durations or differences between timestamps.

🔹 2. Aggregating INTERVALs

Oracle does not allow direct SUM() or AVG() on intervals in older versions. You need to convert intervals into numeric units (like seconds, minutes, or months), aggregate, and then convert back to INTERVAL.

Oracle 23ai has some enhanced support, making this cleaner.

**************************Example*******************
CREATE TABLE work_log (
    emp_id NUMBER,
    duration INTERVAL DAY TO SECOND
);

INSERT INTO work_log VALUES (101, INTERVAL '2 04:30:00' DAY TO SECOND);
INSERT INTO work_log VALUES (102, INTERVAL '1 03:15:30' DAY TO SECOND);
INSERT INTO work_log VALUES (103, INTERVAL '0 08:45:15' DAY TO SECOND);

********************return data**************
SELECT emp_id,
       NUMTODSINTERVAL(SUM(
           EXTRACT(DAY FROM duration) * 86400 +
           EXTRACT(HOUR FROM duration) * 3600 +
           EXTRACT(MINUTE FROM duration) * 60 +
           EXTRACT(SECOND FROM duration)
       ), 'SECOND') AS total_duration
FROM work_log
GROUP BY emp_id;
~~~
#Project
~~~

drop table employee_status;
drop table departments;
-- Employee table with BOOLEAN, INTERVAL, and numeric columns
CREATE TABLE employee_status (
    emp_id NUMBER PRIMARY KEY,
    emp_name VARCHAR2(50),
    is_active BOOLEAN,
    department VARCHAR2(30),
    work_duration INTERVAL DAY TO SECOND
);

-- Department table for join operations
CREATE TABLE departments (
    dept_id NUMBER PRIMARY KEY,
    dept_name VARCHAR2(30),
    location VARCHAR2(30),
    status VARCHAR2(10)  -- Active/Inactive
);

*****___insert the data ******
-- Insert employees
INSERT INTO employee_status (emp_id, emp_name, is_active, department, work_duration)
VALUES 
    (101, 'Alice', TRUE, 'HR', INTERVAL '2 04:30:00' DAY TO SECOND),
    (102, 'Bob', FALSE, 'DEV', INTERVAL '1 03:15:30' DAY TO SECOND),
    (103, 'Charlie', TRUE, 'AI', INTERVAL '0 08:45:15' DAY TO SECOND),
    (104, 'Diana', NULL, 'OPS', INTERVAL '3 01:20:00' DAY TO SECOND);

-- Insert departments
INSERT INTO departments (dept_id, dept_name, location, status)
VALUES 
    (1, 'HR', 'NY', 'Active'),
    (2, 'DEV', 'SF', 'Active'),
    (3, 'AI', 'NY', 'Inactive'),
    (4, 'OPS', 'TX', 'Active');

******** update using the direct join ******
UPDATE employee_status e
SET e.is_active = TRUE
FROM departments d
WHERE e.department = d.dept_name
  AND d.location = 'NY'
  AND d.status = 'Active';

  ****** delet using the direct join method ****
  DELETE FROM employee_status e
USING departments d
WHERE e.department = d.dept_name
  AND d.status = 'Inactive';

  ****Query active employees***
  SELECT emp_id, emp_name, department
FROM employee_status
WHERE is_active;
~~~
