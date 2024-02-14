## Introduction

Handling complex data, like social networks and organizational hierarchies, is increasingly relevant in modern information systems. Modeling such data involves the use of data structures like trees and graphs. These data structures, like the underlying reality they represent, are multilayered and interconnected. Programs written to access this data must therefore navigate through its complex structure to get useful information out of it. This is done using recursive queries.

### Prerequisites

This is a hands-on guide. To benefit from it, you are expected to have prior practical experience with PostgreSQL. To test the examples, it is assumed you already have PostgreSQL running either on a [standalone server](https://www.vultr.com/docs/how-to-install-configure-backup-and-restore-postgresql-on-ubuntu-20-04-lts/) or [as a managed instance](https://www.vultr.com/docs/postgresql-managed-database-guide/).

It is necessary to know [how Common Table Expressions (CTEs) work in PostgreSQL](https://www.postgresql.org/docs/current/queries-with.html). Recursion is based on the use of CTEs. Representing a graph in the form of tables tends to lead to a normalized database design. So, it is useful to understand [how views work](https://www.vultr.com/docs/how-to-use-materialized-views-in-postgresql/), to simplify queries with many `JOIN`s. It is helpful to have an understanding of basic computer science concepts, like recursion and data structures (like [graphs](https://en.wikipedia.org/wiki/Graph_(abstract_data_type)) and [trees](https://en.wikipedia.org/wiki/Tree_(data_structure))).

Some functionality for recursive queries, such as the `CYCLE` keyword, was introduced in PostgreSQL 14.0 (released in 2021). The code samples in this guide are tested on PostgreSQL 14.5. They should be compatible with all PostgreSQL versions higher than 14.0.

### Background

This section is an overview of some basic concepts of tree and graph data structures.

Tree and graph data structures consist of nodes and edges. A node represents a specific entity (e.g. a person, a place, a component). An edge represents the connection between two nodes (e.g. the relationship between two people). Nodes are connected directly (via a single edge) or indirectly (via other nodes and edges). Directly connected nodes are referred to as adjacent nodes. Adjacent nodes are often represented as pairs, like (Node1, Node2). Traversing a graph refers to visiting all or some related nodes of a graph, given a starting node.

In undirected graphs, edges do not have a direction. For example, "A is friends with B" is programmatically the same as "B is friends with A". In directed graphs, the direction of the edge is relevant. An edge from A to B is not the same as an edge from B to A. For example, "A follows B" is not the same as "B follows A". Directed node pairs are ordered: (A, B) is distinct from (B, A). Given a directed edge from A to B, A is the ancestor and B is the descendant. A leaf node is a node with no descendants.

A cycle in a graph occurs when there is a (direct or indirect) path from a node (back) to itself. For example, in directed graphs, a cycle occurs when there is a path back from a descendant node to an ancestor node. Consider a directed graph with nodes A, B, and C. A is an ancestor of B, and B is an ancestor of C. A cycle occurs when C is connected back to A - thus, C becomes an ancestor of A. Note that in this case, a connection from A to C will not make a cycle.

Trees are a subclass of directed acyclic graphs (DAG). Trees have a single "root" node. They are hierarchical data structures - useful to represent things like the reporting structure of an organization (orgchart). In a tree, each ancestor can have multiple descendants. Each descendant has only one ancestor. The depth of a node is the number of edges from that node to the root node.

### Motivation

Consider two examples:

1. Given an orgchart (represented as a tree), you want to find all the people who are in the reporting chain of a particular manager. To do this, you need all the nodes that are (directly or indirectly) descendants of the given node.
1. For a social network (represented as a graph), you want to find all the friends and friends of friends of a person. This is represented by all the nodes which are within 2 degrees of separation from the given node.

To extract this kind of information using regular SQL queries, first find all the nodes adjacent to the given node. Then find all the nodes adjacent to these nodes. Continue this process iteratively until you either:

* reach the leaf nodes, or
* traverse the desired depth.

For a small graph, it is possible, though laborious, to write multiple (sub)queries to do this sequentially. Real-world graphs are large and interconnected with many layers between the root and leaf nodes. In many cases, the depth is not known in advance. Thus a semi-manual iterative approach, as described above, is not practical. Recursive queries solve this problem.

## How recursion works in PostgreSQL

In a regular programming language, a recursive function repeatedly calls itself until the termination condition is met. The output of each iteration serves as the input to the next iteration. The initial value is given to the function and is called the seed value.

In SQL, a recursive query has two parts (sub-queries). The first sub-query is non-recursive. Its result is analogous to the seed value. The second sub-query (recursively) calls the main query. In each iteration, it combines the result with the results from previous iterations of the query.

Because of this structure, CTEs are uniquely suited to implement recursion. CTEs are used to organize complex queries into sub-queries which form the logical building blocks of the (main) query. A recursive CTE is identified by the use of the keyword `RECURSIVE` before the name of the CTE.

### Constraints

To simplify the SQL examples, only *directed graphs* are considered throughout this guide. The initial examples are limited to DAGs (trees are a subclass of DAGs). Later examples show the features of CTEs in PostgreSQL that deal with cyclic graphs.

## Example Data Model

To illustrate the concepts, this guide uses a normalized data model consisting of nodes and edges.

The table `node` has an ID field. The `name` field is a *property* of the node. The table `edge` stores the IDs of two nodes that are adjacent (directly linked) to each other.

In real-world usage, you would also want to have additional columns with properties of the nodes and edges in the respective tables. For example, an edge denoting "is friends with" might have a property like "friends since". An edge between two cities might have properties about the distance and travel time between the cities.

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

Tom is the ancestor of Dick and Sally; Dick is the ancestor of Harry and Jane; Jane of Susan, Mary, and Sam; and Sally is the ancestor of Jack. It is recommended to draw out (on paper) the relationship structure as a tree, using both IDs and names.

### Define a Detailed View

Using a regular (non-recursive) CTE, define a view `nodes_n_edges` which contains the names and IDs of adjacent node pairs:

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

This view is used in later sections to conveniently query the data structure after making changes. Using a view like this is more efficient than repeatedly joining the `nodes` and `edges` tables.

    SELECT * FROM nodes_n_edges;

## Traversing Trees using Recursive CTEs

### Create a Recursive CTE

Suppose you need to get the names of all the people who belong to the branch with root node as Dick. This is analogous to traversing an orgchart to find all the people in Dick's reporting chain.

1. Use the `RECURSIVE` keyword and create a recursive CTE named `rcte`:

		-- incomplete code - do not copy this directly
		WITH RECURSIVE rcte AS (

1. Write the non-recursive sub-query to generate the seed. These are the nodes adjacent to Dick (people directly reporting to Dick):

		-- incomplete code - do not copy this directly
		SELECT node1_id, node1_name, node2_name, node2_id
		FROM nodes_n_edges
		WHERE node1_name = 'Dick'

1. Write the recursive sub-query to `JOIN` the entire query (`rcte`) with the view `nodes_n_edges`:

		-- incomplete code - do not copy this directly
		SELECT ne.node1_id, ne.node1_name, ne.node2_name, ne.node2_id
		FROM rcte
		JOIN nodes_n_edges ne
		ON rcte.node2_id = ne.node1_id

1. Wrap the `UNION` of the two sub-queries inside the (recursive) CTE `rcte` created in Step 1. This is the recursive CTE to traverse the tree.

		WITH recursive rcte as (
			SELECT node1_id, node1_name, node2_name, node2_id
			FROM nodes_n_edges
			WHERE node1_name = 'Dick'

			UNION 

			SELECT ne.node1_id, ne.node1_name, ne.node2_name, ne.node2_id
			FROM rcte
			JOIN nodes_n_edges ne
			ON rcte.node2_id = ne.node1_id
		)
		SELECT node1_name, node2_name 
		FROM rcte ;

The output (of the CTE shown in step 4 above) is the list of node pairs in the sub-tree with its root at Dick.

### Show the Traversal Path

It is helpful to see the path that links two indirectly connected nodes. Since a recursive CTE traverses all the nodes in order, it can also store the path taken to reach each node. To do this, start with the recursive query created in the last section. Initialize an array in the non-recursive sub-query. Each iteration of the recursive sub-query appends the next part of the path to this array.

    WITH RECURSIVE rcte AS (
        SELECT node1_id, node1_name, node2_name, node2_id,
        -- initialize the array
        ARRAY [node1_name, (select('-')), node2_name] AS traversal_path
        FROM nodes_n_edges
        WHERE node1_name = 'Dick'

        UNION 

        SELECT ne.node1_id, ne.node1_name, ne.node2_name, ne.node2_id,
        -- append the path to the array
        rcte.traversal_path||(select(' ; '))||ne.node1_name||(select('-'))||ne.node2_name
        FROM rcte
        JOIN nodes_n_edges ne
        ON rcte.node2_id = ne.node1_id
    )
    SELECT node1_name, node2_name,
        -- select and prettify the text of the path
        REPLACE(REPLACE(traversal_path::text, '"', ''), ',','') AS traversal_path
    FROM rcte ;

### Calculate (and Limit) Traversal Depth

Associating a depth with each relationship helps to 1) understand the depth of the (sub)graph, and 2) limit the query to a specific depth. For example, you might want to do this to find nodes within N degrees of separation from a given node. This is analogous to finding the friends and friends of friends of a person on a social network (graph).

To track the depth of the search, start with the query from the last section. Initialize a `depth` variable to 1 in the non-recursive sub-query. The recursive sub-query increments it on every iteration.

Initialize the non-recursive sub-query to start from the node Tom, to traverse the entire graph (tree). Create a view on the graph traversal query. This view is also reused in later sections.

    CREATE VIEW graph_structure AS
        WITH RECURSIVE rcte AS (
            SELECT node1_id, node1_name, node2_name, node2_id,
            -- initialize a depth variable
            1 AS depth,
            ARRAY [node1_name, (select('-')), node2_name] AS traversal_path
            FROM nodes_n_edges
            WHERE node1_name = 'Tom'

            UNION 

            SELECT ne.node1_id, ne.node1_name, ne.node2_name, ne.node2_id,
            -- increment the depth variable
            rcte.depth + 1,
            rcte.traversal_path||(select(' ; '))||ne.node1_name||(select('-'))||ne.node2_name
            FROM rcte, nodes_n_edges ne
            WHERE rcte.node2_id = ne.node1_id
        )
        SELECT node1_name, node2_name,
            REPLACE(REPLACE(traversal_path::text, '"', ''), ',','') AS traversal_path,
            depth
        FROM rcte ;

Check the traversal path and depths:

    SELECT * FROM graph_structure ;

To limit the depth of the search, add a `WHERE` clause on the `depth` variable:

    WITH RECURSIVE rcte AS (
        SELECT node1_id, node1_name, node2_name, node2_id,
        1 AS depth,
        ARRAY [node1_name, (select('-')), node2_name] AS traversal_path
        FROM nodes_n_edges
        WHERE node1_name = 'Tom'

        UNION 

        SELECT ne.node1_id, ne.node1_name, ne.node2_name, ne.node2_id,
        rcte.depth + 1,
        rcte.traversal_path||(select(' ; '))||ne.node1_name||(select('-'))||ne.node2_name
        FROM rcte, nodes_n_edges ne
        WHERE rcte.node2_id = ne.node1_id
    )
    SELECT node1_name, node2_name,
        REPLACE(REPLACE(traversal_path::text, '"', ''), ',','') AS traversal_path,
        depth
    FROM rcte 
    -- limit the traversal depth
    WHERE depth < 3 ;

## Traversing Acyclic Graphs

The previous section discussed recursive CTEs on trees. Trees are a special case of acyclic graphs. In a tree, each descendant has a single ancestor. In a graph, each descendant can have multiple ancestors.

In the current data, Sam has a single ancestor - Jane. Convert Example Data Model from a tree to a graph by adding an edge from Tom to Sam. So, Sam now has two ancestors - Jane and Tom.

    WITH tom AS (
        SELECT node_id 
        FROM node
        WHERE node_name = 'Tom'
    )
    ,sam AS (
        SELECT node_id 
        FROM node
        WHERE node_name = 'Sam'
    )
    INSERT INTO edge (node1, node2)
    VALUES
        ((SELECT node_id FROM tom), (SELECT node_id FROM sam)) ;

Recheck the view `graph_structure`:

    SELECT * FROM graph_structure ;

The edge from Tom to Sam is now a part of the tree.

The query structure (for trees) used in the previous section also works for acyclic graphs (but not for cyclic graphs). Note the order of nodes inserted in the above query - (Tom, Sam). If this order is reversed, it leads to a *cycle*.

## Traversing Cyclic Graphs

In the data model so far, there are no cycles. Consider the path from Tom to Jane: Tom is an ancestor of Dick and Dick of Jane. Add a cycle in the data by connecting Jane back to Tom:

    WITH jane AS (
        SELECT node_id 
        FROM node
        WHERE node_name = 'Jane'
    )
    ,tom AS (
        SELECT node_id 
        FROM node
        WHERE node_name = 'Tom'
    )
    INSERT INTO edge (node1, node2)
    VALUES
        ((SELECT node_id FROM jane), (SELECT node_id FROM tom)) ;

Recheck the traversal path using the view `graph_structure` created earlier:

    SELECT * FROM graph_structure ;

This query hangs because of an infinite loop. If you are using the terminal, interrupt it manually with `CTRL`+`C`.

The infinite loop is created because every time the search reaches Jane, the path leads back to Tom. This *cycle* continues ad infinitum.

To avoid cycles, use the `CYCLE` clause introduced in PostgreSQL 14.0. The `CYCLE` clause is added at the end of the CTE before the `SELECT`. It defines the columns to check for identifying a cycle. In this case, the presence of a cycle is identified based on the columns `node1` and `node2` of the table `edge`. These columns are referred to as `node1_id` and `node2_id` in the view `nodes_n_edges`.

Start from the query previously written for the view `graph_structure` in the section Calculate and Limit Traversal Depth. Add the `CYCLE` keyword after the sub-queries of the CTE:

    WITH RECURSIVE rcte AS (
        SELECT node1_id, node1_name, node2_name, node2_id,
        1 AS depth,
        ARRAY [node1_name, (SELECT('-')), node2_name] AS traversal_path
        FROM nodes_n_edges
        WHERE node1_name = 'Tom'

        UNION 

        SELECT ne.node1_id, ne.node1_name, ne.node2_name, ne.node2_id,
            rcte.depth + 1,
            rcte.traversal_path||(SELECT(' ; '))||ne.node1_name||(SELECT('-'))||ne.node2_name
        FROM rcte
        JOIN nodes_n_edges ne
        ON rcte.node2_id = ne.node1_id
    )
    -- cycle detection clause
    CYCLE node1_id, node2_id SET is_cycle USING path
    SELECT 
        node1_name, node2_name,
        REPLACE(REPLACE(traversal_path::text, '"', ''), ',','') AS traversal_path,
        depth
    FROM rcte ;

Notice that after the path reaches Tom back from Jane, it (again) goes through the nodes linked to Tom. But it stops before actually reaching Jane (for the second time) - thus avoiding getting into an infinite loop.

## Conclusion

Recursive queries make it possible to model, store, and analyze graph data structures on relational databases. The material in this guide covered the basic usage of recursive queries in PostgreSQL, particularly in the context of directed graphs. Practical applications of directed graphs include modeling social networks based on the "follower" relationship, modeling travel cost and distance to find the most efficient route covering different cities, and so on.

There are two ways of traversing graphs - breadth first and depth first. The [official documentation on recursive CTEs](https://www.postgresql.org/docs/current/queries-with.html#QUERIES-WITH-RECURSIVE) explains how to use the `SEARCH` keyword (introduced in PostgreSQL 14.0) to do this. It also contains a more detailed explanation of how recursive queries work internally. Additionally, it shows how to detect and avoid cycles without actually using the `CYCLE` keyword.

