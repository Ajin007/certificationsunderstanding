# Netflix recommended system
## creation of the table
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

## --------------------------------
Works with functions like:

VECTOR_DISTANCE

VECTOR_DOT_PRODUCT

VECTOR_COSINE_DISTANCE
```

