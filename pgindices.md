# Introduction to Using Indexes in PostgreSQL

## Introduction

When you run a `SELECT` query on a table, the database engine goes through each row sequentially, until it finds the row(s) with the data being searched for. This is called sequential scanning. The order of complexity of a sequential scan is *O(N)*.

Suppose the rows were arranged such that a particular column is sorted (ordered) in ascending order. If you looked up a particular value in this (sorted) column, the engine no longer needs to scan each row sequentially. It is faster to take advantage of the sort order to get to the row(s) where the column value matches the searched value.

This is what an index does. An index is a separate data structure that stores the sorted values of one or more columns. Each value is associated with a pointer to the data of the corresponding row in the table. Using indexes helps to speed up `SELECT` queries. For B-Tree indexes,  which are the most common type of indexes, the order of complexity of an indexed scan is *O(log N)*. PostgreSQL has other types of indexes (e.g., hash indexes) that do not involve sorting column values. Some indexes (e.g., expression indexes) do not store the column values, but the result of a function on those values.

### Pre-requisites

This is a hands-on guide. To benefit from it, you are expected to have prior practical experience with PostgreSQL. To test the examples, it is assumed you already have PostgreSQL running either on a [standalone server](https://www.vultr.com/docs/how-to-install-configure-backup-and-restore-postgresql-on-ubuntu-20-04-lts/) or [as a managed instance](https://www.vultr.com/docs/postgresql-managed-database-guide/).

It is helpful, but not necessary, to be familiar with computer science concepts such as data structures and [computational complexity](https://en.wikipedia.org/wiki/Computational_complexity).

## The `EXPLAIN` Command

[The `EXPLAIN` command in PostgreSQL](https://www.postgresql.org/docs/current/sql-explain.html) is used to estimate (predict) the performance of a query. Using `EXPLAIN` with the `ANALYZE` option executes the query and presents a summary of the execution performance. In the examples in this guide, the `EXPLAIN ANALYZE` command will be used to study the performance of queries before and after indexing. A detailed discussion of `EXPLAIN` is beyond the scope of this guide.

### How to Use

To analyze the performance of a `SELECT` query, prepend the query with `EXPLAIN ANALYZE`:

    /* pseudo-code */
    EXPLAIN ANALYZE
    SELECT ...
    FROM ...
    WHERE ...

To analyze the performance of a data modification query (`INSERT`, `DELETE`, etc.) without modifying the data, wrap the entire query in a transaction block and rollback at the end of it:

    /* pseudo-code */
    BEGIN ; 
    EXPLAIN ANALYZE
    INSERT INTO ... ;
    ROLLBACK ;

### Output of `EXPLAIN`

The example output below is from using `EXPLAIN ANALYZE` on a `SELECT` query that does not use an index.

                                   QUERY PLAN                                                
    ----------------------------------------------------------------------
     Limit  (cost=0.00..353.00 rows=1 width=58) (actual time=2.817..2.838 rows=1 loops=1)
       ->  Seq Scan on person  (cost=0.00..353.00 rows=1 width=58) (actual time=2.802..2.820 rows=1 loops=1)
             Filter: ((name)::text ~~ 'First999%'::text)
             Rows Removed by Filter: 998
     Planning Time: 2.989 ms
     Execution Time: 3.019 ms
    (6 rows)

In addition to other details, it shows the *Planning Time* and *Execution Time* of the query. In later sections, you will compare the execution time of queries with and without indexing.

Notice that the output mentions `Seq Scan on table_name` - this indicates that the query resulted in a sequential scan of the table until it found the searched data. Had the query used an index, it would instead have mentioned `Index Scan using index_name on table_name`.

To enhance readability, the examples in this article will show only part of the `EXPLAIN` output, as below:

                                   QUERY PLAN                                                
    ----------------------------------------------------------------------
     ...
       ->  Seq Scan on person ...
             ...
     Planning Time: 2.989 ms
     Execution Time: 3.019 ms

Omitted text in the output is denoted by ellipses `...`.

### Comparing Results

Query performance, and hence, the output of the `EXPLAIN` function, is dependent on the hardware used. The results in this guide are based on running the queries on a single node (no replicas) instance of Vultr Managed Databases for PostgreSQL on a Cloud Compute server with 1 vCPU and 1 GB DDR4 memory.

PostgreSQL caches query results and query paths to improve the performance on frequently run queries. This tends to bias the results while testing and benchmarking the performance of different indexes. PostgreSQL has no standard way to clear these caches. Running the same (or similar) queries repeatedly can lead to different execution times.

In general, it is difficult to replicate the results, even on the same hardware. Rather than the actual figures, pay attention to the (relative) change in performance with and without indexing, and across different index types.

## Test Data

Create a new table to store (fictitious) name and age data:

    CREATE TABLE person (
        id SERIAL, 
        name VARCHAR(50), 
        age SMALLINT
    ) ;

This query creates a table with an automatically generated ID field, a text field for the name, and an integer field for the age. Later sections will run queries and create indexes on this table. It is advisable to test with a large number of rows to better simulate the conditions of a real-world database. Generate and insert fictitious names and ages of 1 million people:

     INSERT INTO person (name, age)         
     SELECT                                          
        'FirstName' || n || ' LastName' || n AS new_name,
        (MOD(n, 100) + 1) AS new_age
     FROM generate_series(1,1000000) AS n ;  

Note the syntax for series generation and string concatenation in the above query.

Note also that inserting a million rows can take some time - a few seconds or more, depending on the hardware of your system. On the test server, it takes up to 30 seconds.

Check the inserted data:

    SELECT * FROM person ORDER BY random() LIMIT 5 ;

This shows five random rows from the table. Observe the form of the inserted data.

## B-Tree Indexes

B-Tree indexes are the default type of index created by PostgreSQL. They are based on the [B-Tree data structure](https://en.wikipedia.org/wiki/B-tree). This section shows the performance of `SELECT`, `INSERT`, and `DELETE` queries on the test table. Run each query without and with the index, and compare the results.

### `SELECT` Queries

#### Performance Without Indexing

Write a query that does a `SELECT` based on the value of the `age` column. Check its performance using `EXPLAIN`:

    EXPLAIN ANALYZE
    SELECT * FROM person WHERE age BETWEEN 5 AND 10 ;

Note the execution time:

                                   QUERY PLAN                                                
    ----------------------------------------------------------------------
    ...
      ->  Parallel Seq Scan on person ...
    ...
    Planning Time: 0.245 ms                                                                                                     
    Execution Time: 1058.236 ms                                                                                                 

#### Index Creation

The general syntax for creating a new index looks like this:

    /* pseudo-query to show the syntax */
    CREATE INDEX my_index_name ON my_table_name USING index_type (my_column_name) ;

`index_type` denotes the type of index - B-Tree, Hash, GIN, BRIN, etc.

##### Create B-Tree Index

To create a B-Tree index, specify the name of the table and the column on which the index is to be created. Since it is the default type of index, you do not need to explicitly specify the index type as B-Tree.

    CREATE INDEX idx_age_btree ON person (age) ;

This creates a B-Tree index `idx_age_btree` on the column `age`:

#### Performance With the Index

Run again the same `SELECT` query and check its performance using `EXPLAIN`.

    EXPLAIN ANALYZE
    SELECT * FROM person WHERE age BETWEEN 5 AND 10 ;

The result looks like this:

                                   QUERY PLAN                                                
    ----------------------------------------------------------------------
    Index Scan using idx_age_btree on person  ...
      ...
    Planning Time: 0.974 ms                                                                                                           
    Execution Time: 271.304 ms                                                                                                        

Using the index results in a lower execution time.

Notice that the query plan specifies `Seq Scan` (sequential scan) in the first case and `Index Scan` in the second case. Notice also that the planning time is higher when using the indexes. This is because the planner now has more options to consider while planning the execution of the query.

#### Drop the Index

Drop the index before testing the performance of other queries in subsequent sections.

    DROP INDEX idx_age_btree ;

Dropping an index is a blocking operation - no other queries can execute while the index is being deleted. To allow other queries to be executed while the index is being deleted, use the `CONCURRENTLY` option:

    DROP INDEX CONCURRENTLY idx_age_btree ;

### INSERT Queries

#### `INSERT` Performance Without Indexing

Check the performance of inserting a few rows into the table (without an index):

     BEGIN ;
     EXPLAIN ANALYZE
     INSERT INTO person (name, age)         
     SELECT                                          
        'FirstName' || n || ' LastName' || n AS new_name,
        (MOD(n, 100) + 1) AS new_age
     FROM generate_series(1,10) AS n ;  
     ROLLBACK ;

The output looks like this:

                                   QUERY PLAN                                                
    ----------------------------------------------------------------------
    Insert on person  ...
    ...
    Planning Time: 0.303 ms                                                                                                 
    Execution Time: 0.454 ms                                                                                                

#### `INSERT` Performance Using the Index

Recreate the same index as before:

    CREATE INDEX idx_age_btree ON person (age) ;

Check again the performance of the same insertion query written earlier:

     BEGIN ;
     EXPLAIN ANALYZE
     INSERT INTO person (name, age)         
     SELECT                                          
        'FirstName' || n || ' LastName' || n AS new_name,
        (MOD(n, 100) + 1) AS new_age
     FROM generate_series(1,10) AS n ;  
     ROLLBACK ;

The output now looks like this:

                                   QUERY PLAN                                                
    ----------------------------------------------------------------------
    Insert on person ...
    ...
    Planning Time: 0.183 ms                                                                                                 
    Execution Time: 6.898 ms                                                                                                

The execution time is higher when the index is present. This is because the index needs to be updated after the new data is inserted. The more indexes a table has, and the more complex the indexes, the longer data modification queries take.

Drop the index before testing the performance of other queries in subsequent sections.

    DROP INDEX idx_age_btree ;

### DELETE Queries

#### `DELETE` Performance Without Indexing

Deleting data from a table involves two actions - 1) identify the *location* of the data to be deleted, and 2) delete the data. Using indexes speeds up the first action but slows down the second action (because the index needs to be updated).

Check the deletion performance when there are no indexes:

    BEGIN ;
    EXPLAIN ANALYZE

    DELETE FROM person
    WHERE age = 42 ;
    ROLLBACK ;

The `DELETE` query is based on the value of the `age` column.

The output looks like this:

                                   QUERY PLAN                                                
    ----------------------------------------------------------------------
    Delete on person ... 
      ->  Seq Scan on person ... 
    ...
    Planning Time: 0.298 ms                                                                                           
    Execution Time: 3058.996 ms                                                                                       

#### `DELETE` Performance Using the Index

As before, recreate the index on the `age` column:

    CREATE INDEX idx_age_btree ON person (age) ;

Check again the performance of the deletion query written earlier:

    BEGIN ;
    EXPLAIN ANALYZE

    DELETE FROM person
    WHERE age = 42 ;
    ROLLBACK ;

The output looks like this:

                                   QUERY PLAN                                                
    ----------------------------------------------------------------------
    Delete on person ...
      ->  Index Scan using idx_age_btree on person ...
    ...
    Planning Time: 0.383 ms                                                                                                              
    Execution Time: 93.838 ms                                                                                                            

The deletion performance is improved (execution time is less) while using the index.

Drop the index before testing the performance of other queries in subsequent sections.

    DROP INDEX idx_age_btree ;

## `UNIQUE` indexes

`UNIQUE` indexes are used to ensure (*enforce*) that a column can only contain unique values. For example, you might want to enforce a uniqueness constraint on a column containing the email IDs used to create user accounts. Uniqueness constraints can also be enforced across multiple columns. In this case, the combination of values across all the columns has to be unique. The individual columns themselves can contain duplicate values.

To enforce a uniqueness constraint on the `name` column in the example table, define a `UNIQUE INDEX` on that column:

    CREATE UNIQUE INDEX idx_name_unique on person(name) ;

The original data insertion query (in the Test Data section) inserted names `FirstName1 LastName1`, `FirstName2 LastName2`, and so on, until `FirstName1000000 LastName1000000`. Try to insert a few duplicate values for the `name` field :

     INSERT INTO person (name, age)         
     SELECT                                          
        'FirstName' || n || ' LastName' || n AS new_name,
        (MOD(n, 100) + 1) AS new_age
     FROM generate_series(1,10) AS n ;  

The above query again tries to insert names `FirstName1 LastName1`, `FirstName2 LastName2`, and so on, until `FirstName10 LastName10`, which are already present in the table. So, you get an error like this:

    duplicate key value violates unique constraint "idx_name_unique"

The presence of the `UNIQUE INDEX` on the `name` column prevents the creation of duplicate names.

Note that if you try to define a uniqueness constraint on a column that already contains duplicate values, it will lead to an error.

### Multicolumn `UNIQUE` Indexes

Defining multicolumn uniqueness constraints works similarly to the single-column case. The general syntax for creating a multicolumn `UNIQUE` index is:

    CREATE UNIQUE INDEX my_unique_index_name on my_table_name(my_column_name1, my_column_name2, ...) ;

## Multicolumn Indexes

Multicolumn indexes can improve the performance of queries that search across multiple columns. For example, a query of the form `SELECT * FROM my_table_name WHERE my_column1 = value1 AND my_column2 = value2` can benefit from a multicolumn index defined on columns `my_column1` and `my_column2`. Additionally, for queries spanning multiple columns,  PostgreSQL is also able to leverage indexes on the individual columns.

To understand multicolumn indexes, write a query that spans two columns. Test the performance of the query in 4 scenarios:

1. without using an index
1. using a single-column index on one of the columns in the query
1. using two individual single-column indexes on both columns in the query
1. using a multicolumn index on both columns in the query

The examples in this section are based on the (default) B-Tree index type.

### Multicolumn Query Without Indexing

Write and test a query that depends on the values of the columns `age` and `id`:

    EXPLAIN ANALYZE
    SELECT * FROM person 
    WHERE id BETWEEN 990000 AND 999999 AND age = 100 ;

The result looks like this:

                                   QUERY PLAN                                                
    ----------------------------------------------------------------------
    ...
      ->  Parallel Seq Scan on person ...
    ...
    Planning Time: 0.665 ms                                                                                                   
    Execution Time: 1600.011 ms                                                                                               

### Multicolumn Query Using One Single-column Index

Create an index on (only) the `age` column:

    CREATE INDEX idx_age_btree ON person(age) ;

Run and test the same query as before:

    EXPLAIN ANALYZE
    SELECT * FROM person 
    WHERE id BETWEEN 990000 AND 999999 AND age = 100 ;

The result looks like this:

                                   QUERY PLAN                                                
    ----------------------------------------------------------------------
    Index Scan using idx_age_btree on person ...
    ...
    Planning Time: 0.514 ms                                                                                                      
    Execution Time: 24.351 ms                                                                                                    

The execution time is less when using the index. Using even one single-column index can help speed up multicolumn queries.

This example creates an index on one of the columns (`age`) used in the query. You can also try having an index on the other column (`id`).

### Multicolumn Query Using Multiple Single-column Indexes

You already have an index on the `age` field. Create an index on the `ID` field:

    CREATE INDEX idx_id_btree ON person(id) ;

Now, you have two single-column indexes. Run and test the same query that depends on the values of both `age` and `id` columns:

    EXPLAIN ANALYZE
    SELECT * FROM person 
    WHERE id BETWEEN 990000 AND 999999 AND age = 100 ;

The result looks like this:

                                   QUERY PLAN                                                
    ----------------------------------------------------------------------
    ...
            ->  Bitmap Index Scan on idx_age_btree  ...
                  ...
            ->  Bitmap Index Scan on idx_id_btree ...
                  ...
    Planning Time: 0.618 ms                                                                                                            
    Execution Time: 2.601 ms                                                                                                           

The result mentions that both indexes were used in the query. The execution time is also less compared to the single index example.

Drop both the single-column indexes:

    DROP INDEX idx_id_btree, idx_age_btree ;

### Multicolumn Query Using a Multicolumn Index

Now, create a multicolumn index on the `id` and `age` columns:

    CREATE INDEX idx_age_id_btree ON person(id, age) ;

Check the performance

    EXPLAIN ANALYZE
    SELECT * FROM person 
    WHERE id BETWEEN 990000 AND 999999 AND age = 100 ;

The output looks like this:

                                   QUERY PLAN                                                
    ----------------------------------------------------------------------
    Index Scan using idx_age_id_btree on person ...
    ...
    Planning Time: 0.677 ms                                                                                                      
    Execution Time: 1.038 ms                                                                                                     

The performance using the multicolumn index is better than in other cases. Note that due to this being a simplistic example, and due to the effects of query path caching, you may or may not be able to replicate these result patterns.

Multicolumn indexes are more resource-expensive than single-column indexes. Before deploying a multicolumn index, it is advisable to test the four different scenarios as shown in this section. In some cases, one or two single-column indexes can be more efficient than multicolumn indexes.

## Basic Administration Information

PostgreSQL has many functions to get information on the indexes themselves.

### List Indexes on a Table

To know the indexes created on a table, query the builtin view `pg_indexes`:

    SELECT * FROM pg_indexes WHERE tablename = 'my_table' ;

If you are using the `psql` command line, you can also check the description of the table by entering `\d my_table` at the `psql` prompt. The table description includes the indexes created on that table.

### Index and Table Size

To check the size of an index, call the `pg_table_size` function with the name of the index:

    SELECT pg_table_size('my_index_name') ;

The above command returns the disk space (in bytes) taken by the index `my_index_name`.

The same command is also used to get the space used by a table.

    SELECT pg_table_size('my_table_name') ;

The above command returns the disk space (in bytes) taken by the table `my_table_name`. It does not include the space used by indexes on the table.

To get the total disk space used by all indexes on a table, use `pg_indexes_size`:

    SELECT pg_indexes_size('my_table_name') ;

## Conclusion

Most of the examples in this article are based on B-Tree indexes. PostgreSQL has many other types of indexes, like [hash indexes, which are useful for equality comparisons](https://www.postgresql.org/docs/current/indexes-types.html#INDEXES-TYPES-HASH), [BRIN indexes, which are useful for date and time ranges](https://www.postgresql.org/docs/current/indexes-types.html#INDEXES-TYPES-BRIN), and [GIN indexes, which are useful for JSON data and text searches](https://www.postgresql.org/docs/current/indexes-types.html#INDEXES-TYPES-GIN). It is also possible to declare [indexes on expressions](https://www.postgresql.org/docs/current/indexes-expressional.html). [Covering indexes](https://www.postgresql.org/docs/current/indexes-index-only-scans.html), in certain situations, offer further performance enhancement over regular indexes.

It is important to understand the right index for each use case, depending on 1) the type of data being queried and 2) the type of queries expected to be run on that data. For using indexes in practice, it is important to:

1. Decide the type of index based on 1) the data type of the column being indexed, and 2) the query type that you want to optimize
1. Benchmark and test the performance of the specific query type (e.g. `SELECT` queries) before and after creating the index.
1. Benchmark and test the performance of other relevant queries (e.g. data insertion queries) before and after creating the index.

The presence of an index does not guarantee it will be used. The [PostgreSQL query planner](https://www.postgresql.org/docs/current/planner-optimizer.html) decides which index, if any, to choose for a given query.

### Trade-offs

Although the right index speeds up `SELECT` queries, it also slows down `INSERT` and `UPDATE` queries. Here are a few ways to address this:

1. On production databases, it is common practice to have read-only copies of tables with "read-heavy" workloads. It is advisable to segregate tables into "read-heavy" and "write-heavy" during the database design process.
1. [Materialized views](https://www.vultr.com/docs/how-to-use-materialized-views-in-postgresql/) are a preferred approach when the database is (partially) normalized and most queries already involve joins. It is also possible to create indexes on materialized views in PostgreSQL.

Indexes also take up some amount of space. The amount of space the index consumes depends on the type of index and the size of the table.
