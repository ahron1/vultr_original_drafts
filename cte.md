# Introduction to Common Table Expressions in PostgreSQL

## Motivation

Complex SQL queries involve multiple subqueries. Common Table Expressions (CTEs) are used to organize the individual subqueries as logical building blocks of the (main) query. The resulting code is more readable and better organized. These building blocks can also be reused in different places throughout the query.

A CTE starts with the keyword `WITH`. Because of their syntax, CTEs are sometimes known as *WITH queries*. In its simplest form, a CTE looks like this:

    -- pseudocode
    WITH my_cte AS (
        SELECT ...
        FROM ...
        WHERE ...
    )
    SELECT ... 
    FROM my_cte ;
    -- or other operations like JOINS

The CTE, `my_cte` temporarily holds the result of the subquery within the parentheses. This result set is valid only within the scope of the query. Hence, CTEs are also known as *inline views*.

## Prerequisites

To benefit from this guide, you are expected to have some prior practical experience with PostgreSQL. This is a hands-on guide, it is advisable to follow along the examples. It is assumed you already have PostgreSQL running either on a [standalone server](https://www.vultr.com/docs/how-to-install-configure-backup-and-restore-postgresql-on-ubuntu-20-04-lts/) or [as a managed instance](https://www.vultr.com/docs/postgresql-managed-database-guide/).

PostgreSQL has supported CTEs since version 8.4 (released in 2009). The code samples in this guide are tested on PostgreSQL 14.5. They should be compatible with all recent PostgreSQL versions.

## Example Data Model - I

To illustrate the concepts, this guide uses a simple normalized data model consisting of two tables: `node` and `edge`. Hypothetically, if the nodes represent people, the edges represent a relationship between two people, e.g. "A is friends with B", "A follows B", "A reports to B", and so on.

The table `node` has an ID field and a name field. The table `edge` stores the IDs of two nodes that are adjacent (directly linked) to each other.

### Set up the Tables

Create a table `node` with two columns - `node_id`, `node_name`:

    CREATE TABLE node (
        node_id INTEGER PRIMARY KEY,
        node_name VARCHAR NOT NULL
    );

Create a table `edge` with two columns - `node1` and `node2`, both referencing foreign keys to the table `node`:

    CREATE TABLE edge (
        node1 INTEGER REFERENCES node (node_id),
        node2 INTEGER REFERENCES node (node_id),
        PRIMARY KEY (node1, node2)
    );

### Add Test Data

Insert test data into each of the tables. Create a few nodes (representing people):

    INSERT INTO node (node_id, node_name) 
    VALUES (1, 'Tom'), (2, 'Dick'), (3, 'Harry'), (4, 'Jane'), 
        (5, 'Susan'), (6, 'Mary'), (7, 'Sam'), (8, 'Sally'), (9, 'Jack') ;

Create edges to define hypothetical relationships between the nodes:

    INSERT INTO edge (node1, node2) 
    VALUES (1, 2), (1, 8), (2, 3), (2, 4), (4, 5), (4, 6), (4, 7), (8, 9) ;

Tom is linked with Dick and Sally; Dick with Harry and Jane; Jane with Susan, Mary, and Sam; and Sally with Jack. It is recommended to draw out (on paper) the relationship structure as a tree, using both IDs and names.

## CTEs Using `SELECT` Queries

### Start with Views

Create a view `my_view` that shows the names of nodes (people) and the IDs of adjacent (i.e. connected via a single edge) nodes:

    CREATE VIEW my_view AS
        SELECT n.node_name, e.node2
        FROM node n
        JOIN edge e 
        ON n.node_id = e.node1;

`JOIN` this view with the table `node` to show the names of pairs of adjacent nodes:

    SELECT 
        my_view.node_name AS node1_name, 
        node.node_name AS node2_name
    FROM my_view 
    JOIN node
    ON my_view.node2 = node.node_id;

This outputs two columns with the names of adjacent node pairs.

### CTE instead of View

A CTE creates a named subquery and accesses it later within the primary statement.

Rewrite the previous example using a CTE. Create a CTE `my_cte` defined similarly to the view `my_view`. Join it with the table `node` to get the names of adjacent node pairs:

    WITH my_cte AS (
        SELECT n.node_name, e.node2
        FROM node n
        JOIN edge e 
        ON n.node_id = e.node1
    )
    SELECT 
        my_cte.node_name AS node1_name, 
        node.node_name AS node2_name
    FROM my_cte 
    JOIN node 
    ON my_cte.node2 = node.node_id;

Notice that the CTE's definition(the `SELECT` query) inside the parentheses is the same as the definition of `my_view` in the previous section. The output of this query should be the same as in the previous example using the view.

The subquery within the parentheses can be any legitimate SQL query. The subquery as well as the primary statement can be based on `SELECT`, `INSERT`, `UPDATE`, or `DELETE`. This section shows how to use `SELECT` queries in a CTE. A later section is devoted to data modification queries, like `INSERT`.

## Multiple CTEs in the Same Query

To have multiple CTEs in the same query, use a single `WITH` keyword in the entire query and separate the different CTE definitions using commas. Starting with the previous example, move the second part of the query (after the definition of `cte1`) into another CTE:

    WITH
    cte1 AS (
        SELECT n.node_name, e.node2
        FROM node n
        JOIN edge e 
        ON n.node_id = e.node1
    )
    ,cte2 AS (
        SELECT 
            cte1.node_name AS node1_name, 
            node.node_name AS node2_name
        FROM cte1 
        JOIN node 
        ON cte1.node2 = node.node_id
    )
    SELECT * FROM cte2;

The output should be the same as in the last section.

Notice that the second CTE (`cte2`), accesses the results of the first CTE (`cte1`) within its definition.

### Nested CTEs

You can also use a CTE inside the definition of another CTE. As a demonstration (this does not give any additional practical benefit in this specific case), wrap together both CTEs of the previous example inside a third CTE:

    WITH 
    cte3 AS (
        WITH
        cte1 AS (
            SELECT n.node_name, e.node2
            FROM node n
            JOIN edge e 
            ON n.node_id = e.node1
        )
        ,cte2 AS (
            SELECT 
                cte1.node_name AS node1_name, 
                node.node_name AS node2_name
            FROM cte1 
            JOIN node 
            ON cte1.node2 = node.node_id
        )
        SELECT * FROM cte2
    )
    SELECT * FROM CTE3;

## Views on CTEs

Defining a view over a CTE is the same as creating a view over a regular query. Define a view `nodes_n_edges` which contains the names and IDs of adjacent node pairs:

    CREATE VIEW nodes_n_edges AS
        WITH cte1 AS (
            SELECT n.node_id, n.node_name, e.node2
            FROM node n
            JOIN edge e 
            ON n.node_id = e.node1
        )
        SELECT 
            cte1.node_id AS node1_id, 
            cte1.node_name AS node1_name, 
            node.node_name AS node2_name,
            node.node_id AS node2_id
        FROM cte1 
        JOIN node 
        ON cte1.node2 = node.node_id;

This view is used in later sections to conveniently (re-)check the data structure after making changes.

    SELECT * FROM nodes_n_edges;

## CTEs with Data Modification Queries

Suppose you need to add a new node. The new node represents Jill and is linked to Jack's node.

Add to the table `node` a new entry (for a new node) with ID as 10 and the name Jill:

    INSERT INTO node (node_id, node_name) VALUES (10, 'Jill') ;

Use two CTEs to first fetch the IDs of Jack and Jill, given their names. Then use these IDs in an `INSERT` query to create an entry in the `edge` table:

    WITH 
    node1_id AS (
        SELECT node_id 
        FROM node
        WHERE node_name = 'Jack'
    )
    ,node2_id AS (
        SELECT node_id
        FROM node
        WHERE node_name = 'Jill'
    )
    INSERT INTO edge (node1, node2)
    VALUES
        ((SELECT node_id FROM node1_id), (SELECT node_id FROM node2_id)) ;

(Re-)Check in the `nodes_n_edges` view that Jill is added and that Jack is linked to Jill:

    SELECT * FROM nodes_n_edges;

### CTE with `INSERT`...`RETURNING`

While creating a new node in the previous example:

1. The `ID` field (in the table `node`) was manually chosen and inserted.
1. The entry in the other table (`edge`) was inserted in a subsequent query.

In practice, it often happens that IDs are automatically generated when the new row is inserted. The automatically generated ID is then used to create new rows in other tables. All this needs to happen in a single transaction. This is difficult using the type of queries shown in the previous example. This is where the `RETURNING` keyword comes in. The `RETURNING` keyword in PostgreSQL returns values from data modification queries.

Create another model with a table where the ID field is automatically generated.

#### Example Data Model - II

Consider a data model consisting of two tables: `business` and `address`. The table `business` has two columns `business_id`, and `business_name`. `business_id` is an automatically generated serial number. The table `address` has two columns - `business_id` (a foreign key to the `business` table) and `address`, character field to store the address of the business.

For every new business account, an entry is inserted in the table `business`. The table auto-generates the ID. The `INSERT`...`RETURNING` query returns the auto-generated ID of the new business. This ID is used in the next subquery to create a new entry in the `address` table. CTEs with the `INSERT...RETURNING` clause provide a convenient way of handling all this in a single query.

Create the table `business`:

    CREATE TABLE business (
        business_id SERIAL PRIMARY KEY, 
        business_name VARCHAR(20)
    );

Create the table `address`:

    CREATE TABLE address (
        business_id INTEGER PRIMARY KEY REFERENCES business (business_id), 
        business_address VARCHAR (200)
    );

#### Query with Auto-generated ID

In the query below, the first CTE inserts a new row in the `business` table and *returns* the auto-generated ID. The second CTE uses this returned ID to create a new row in the `address` table.

    WITH 
        new_business AS (
            INSERT INTO business (business_name)
            VALUES ('City Bakery')
            RETURNING business_id, business_name
        ),
        new_address AS (
            INSERT INTO address (business_id, business_address)
            SELECT business_id, 'address line 1' FROM new_business
            RETURNING business_id, business_address
        )
    SELECT 
        new_business.business_id, new_business.business_name, new_address.business_address
    FROM new_address
    JOIN new_business
    ON new_business.business_id = new_address.business_id ;

In this query, the `business_name` and `address` fields are hard-coded values. In practice, they are given to the query as inputs (for example, by a user creating a new account on the web front-end). The above query returns the auto-generated ID together with the name and address of the new business.

## Next Steps

This guide discussed regular (non-recursive) CTEs. They are useful to enhance code structure and readability. In tables with automatically generated fields (e.g. IDs), CTEs are useful to insert new rows within a single query. The [PostgreSQL documentation](https://www.postgresql.org/docs/current/queries-with.html) shows more complex and interesting use cases of CTEs and explains the inner workings of many features.

After learning the material in this guide, a good next step is to learn about [recursive CTEs](https://www.postgresql.org/docs/current/queries-with.html#QUERIES-WITH-RECURSIVE). Recursive CTEs are uniquely useful for (recursively) querying hierarchical data structures (like trees) and more complex data structures (like graphs). These data structures have practical applications like modeling organizational trees and social networks.
