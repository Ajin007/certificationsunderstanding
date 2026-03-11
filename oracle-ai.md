# Netflix recommended system
## creation of the table
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
