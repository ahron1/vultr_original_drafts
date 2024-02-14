# How to Use Materialized Views in PostgreSQL

## Motivation

### Views

Given a complex SQL query, it is impractical to rewrite the entire query every time its results are needed. Views solve this problem. A **view** is a named (pre-defined) query and a pseudo-table with the output of that query.

The code to make a view based on a query looks like this:

    -- pseudocode
    CREATE VIEW my_view AS
        SELECT ...
            FROM ... JOIN ... ON ...
            WHERE ... AND ... 
            ORDER BY ...
            LIMIT ...

Essentially, the query is prepended with `CREATE VIEW my_view AS`. This creates a new view, `my_view`; query it as if it were a regular table:

    -- pseudocode
    SELECT ...  
        FROM my_view 
        WHERE ...

Thus, you can access the results of the query using just the view, without having to write out the entire query.

### Where Views Fall Short

A view is partially analogous to a "function" in a programming language - a short name for a complex query. It helps with the user experience and readability of the code.

But under the surface, every time the view is accessed, the database converts the view to the full query and (re-)evaluates it before presenting the output. Recomputing a complex query every time is inefficient and leads to no performance gains for the database. On the contrary, repeatedly executing complex queries on large tables degrades the performance. This is where **materialized views** come in.

## Materialized Views

Materialized views solve this problem by storing (caching) the output of a named (pre-defined) query in a persistent data structure similar to a table. You run `SELECT` queries and create indices on them as if they were a regular table. It is also possible to construct materialized views based on queries on other materialized views.

Materialized views are available in PostgreSQL, Oracle Database, SQL Server, and a few other database engines. This feature is not available on MySQL.

### Prerequisites

Basic familiarity with the PostgreSQL database is necessary to benefit from this guide. It is assumed you have some experience with PostgreSQL basics - installing the software, creating a new database, creating tables, standard queries, and so on. For the SQL examples, it is assumed that you already have a running PostgreSQL instance set up on either [Ubuntu](https://www.vultr.com/docs/how-to-install-configure-backup-and-restore-postgresql-on-ubuntu-20-04-lts/), [FreeBSD](https://www.vultr.com/docs/how-to-install-postgresql-on-freebsd-12-2/), [CentOS](https://www.vultr.com/docs/install-postgresql-on-centos-7/) or its [successors](https://www.vultr.com/docs/how-to-choose-the-best-operating-system-for-a-vultr-cloud-server/), or that you are using a [managed database service](https://www.vultr.com/docs/postgresql-managed-database-guide/).

The SQL examples in this guide are tested on PostgreSQL 14.5 running on FreeBSD 13.1-RELEASE. They should be compatible with all recent versions of PostgreSQL running on all recent Operating Systems.

## Practical Examples

### Set up Test Tables

Before creating a materialized view, set up two test tables and populate them with data.

Create a table `product`:

    CREATE TABLE IF NOT EXISTS product (
        product_id INTEGER PRIMARY KEY, 
        name VARCHAR(20) NOT NULL, 
        price SMALLINT NOT NULL
    );

Create a table `orders`:

    CREATE TABLE IF NOT EXISTS orders (
        order_id INTEGER, 
        product_id INTEGER REFERENCES product (product_id),
        PRIMARY KEY (order_id, product_id)
    );

Check the descriptions of the created tables:

    \d orders

Insert some rows with dummy data into the `product` table:

     INSERT INTO product (product_id, name, price) VALUES (1, 'Floppy Disk Drive', 40);
     INSERT INTO product (product_id, name, price) VALUES (2, 'Oculus Quest', 400);

Insert dummy data into the `orders` table:

    INSERT INTO orders (order_id, product_id) VALUES (1, 1);
    INSERT INTO orders (order_id, product_id) VALUES (1, 2);
    INSERT INTO orders (order_id, product_id) VALUES (2, 1);

Check the data in the tables:

    SELECT * FROM orders;

    SELECT * FROM product;

### Create a Materialized View

Create a materialized view on a join query:

    CREATE MATERIALIZED VIEW mv_products_orders
    AS
    SELECT 
        p.product_id, 
        o.order_id, 
        p.name, 
        p.price 
    FROM 
        product p 
    JOIN 
        orders o 
    ON 
        p.product_id = o.product_id;

This creates a materialized view `mv_products_orders`, and populates it based on the data in the underlying table(s) as at the time of creation of the materialized view. By default, the column names of the materialized view derive from the column names of the underlying tables.

Check the definition of the newly created materialized view:

    \d mv_products_orders

Check the data in `mv_products_orders`:

    SELECT * FROM mv_products_orders;

To rename a materialized view, use the `ALTER` command:

    ALTER MATERIALIZED VIEW mv_products_orders RENAME TO my_mv;

### Create an Index on the Materialized View

Indices on materialized views have the same benefits as they do on regular tables - they help in fast lookups. In particular, a common way of refreshing (discussed in a later section) materialized views requires the use of indices. You can define an index on any column(s) of a materialized view.

    CREATE UNIQUE INDEX product_order ON mv_products_orders (order_id, product_id);

Check that the description of the materialized view now includes the index:

    \d mv_products_orders

### (Materialized) Views on (Materialized) Views

It is possible to construct materialized views based on other materialized views.

    CREATE MATERIALIZED VIEW my_mv
    AS
    SELECT 
        * from mv_products_orders
        limit 2;

Similarly, it is also possible to create views based on materialized views, and materialized views based on views.

## Refreshing (Updating) Materialized Views

PostgreSQL does not automatically refresh materialized views. This means, by default, the data in the materialized view becomes outdated when the underlying tables are updated. You need to update the materialized view manually or configure a system to do it automatically.

Add a new row to the `orders` table:

    INSERT INTO orders (order_id, product_id) VALUES (2, 2);

Re-check the materialized view:

    SELECT * FROM mv_products_orders;

The output is still the same as before.

### Manual Refresh

The `REFRESH` command is used to refresh the contents of a materialized view:

    REFRESH MATERIALIZED VIEW mv_products_orders;

Check that the materialized view now includes the newly added data to the `orders` table:

    SELECT * FROM mv_products_orders;

Doing a refresh discards the old contents and recreates the materialized view. Note that it is not possible to query the materialized view while it is being refreshed in this way. The refresh operation places a lock on the materialized view and blocks even `SELECT` queries on it. This lock is held until the end of the (refresh) transaction.

#### The `CONCURRENTLY` Option

Refreshing with the `CONCURRENTLY` option solves this problem.

    REFRESH MATERIALIZED VIEW CONCURRENTLY mv_products_orders;

With the `CONCURRENTLY` option, the database does not block `SELECT` queries on the materialized view while it is being refreshed. When this option is specified, internally, it creates a temporary data structure with the new results of the materialized view's query. The old and new results are compared and the changes are applied to the original materialized view using `UPDATE` and `INSERT` operations.

Note that in order to refresh concurrently, the materialized view must contain at least one column based unique index. When you try to use this on a materialized view that does not have a unique index, it throws an error:

    ERROR:  cannot refresh materialized view "public.mv_products_orders" concurrently
    HINT:  Create a unique index with no WHERE clause on one or more columns of the materialized view. 

Note also that it is possible to run only one refresh operation on a materialized view at a time (even with the `CONCURRENTLY` option).

#### Trade-off

If an update (refresh) involves a lot of new data, the speed of refresh is faster when the `CONCURRENTLY` option is **not** used. This is because of all the comparisons and update operations involved in a concurrent refresh.

**Practical Tip:** In general, it is helpful to periodically vacuum a database to clean up unused data structures and free up space. This is especially relevant with a concurrent refresh because this operation involves creation of temporary data structures. It is advisable to do the vacuuming after the refresh. [Vacuuming](https://www.postgresql.org/docs/current/sql-vacuum.html) is an extensive topic in itself and out of the scope of this guide.

### Automatic Refresh

As of November 2022, PostgreSQL has no features for automatically refreshing materialized views. It is, however, possible to set up automatic refreshes using other tools.

#### Cron Jobs

A common approach to automatically refresh materialized views is by [using cron jobs](https://www.vultr.com/docs/how-to-use-the-cron-task-scheduler/):

    15 * * * * psql -d name_of_your_database -c "REFRESH MATERIALIZED VIEW CONCURRENTLY mv_products_orders"

Adding this line to the crontab of user `postgres` will call the `psql` command every 15 minutes and pass to it as parameters the name of the database and the SQL command to refresh the materialized view.

For the `psql` command above, note that if you did not explicitly create or connect to a specific database, by default, queries are executed in the `postgres` database.

**Practical Tip:** Since only one refresh operation can run at a time, it is important to have some idea about how long a refresh operation takes before scheduling cron jobs for it.

#### Triggers

It is also possible to use [triggers](https://www.postgresql.org/docs/current/sql-createtrigger.html) to update materialized views. To do this, create a function that refreshes the materialized view. On those tables whose data goes into the materialized view, set a trigger to call this function after `INSERT`, `UPDATE`, and `DELETE` operations.

Create a [PL/pgSQL - the SQL Procedural Language](https://www.postgresql.org/docs/current/plpgsql.html) function that refreshes the materialized view:

    CREATE OR REPLACE FUNCTION mv_refresh()
    RETURNS trigger LANGUAGE plpgsql AS $$
    BEGIN
        REFRESH MATERIALIZED VIEW CONCURRENTLY mv_products_orders;
        RETURN NULL;
    END;
    $$;

Create a trigger that calls this function when certain operations (`INSERT`, `UPDATE`, and `DELETE`) are run on the `orders` table:

    CREATE TRIGGER mv_trigger 
    AFTER INSERT OR UPDATE OR DELETE
    ON orders
    FOR EACH STATEMENT EXECUTE PROCEDURE mv_refresh();

Check that the definition of the `orders` table includes the trigger:

    \d orders

Similarly, add a trigger to call the `mv_refresh()` function when the data in the `products` table changes.

Delete a row from the orders table:

    DELETE FROM orders WHERE order_id = 1 AND product_id = 2;

Check that the materialized view no longer includes the deleted row:

    SELECT * FROM mv_products_orders;

## Dropping Materialized Views

Dropping a materialized view is similar to dropping a regular view.

    DROP MATERIALIZED VIEW mv_products_orders;

To drop the materialized view along with all other objects that depend on it, use the `CASCADE` option:

    DROP MATERIALIZED VIEW IF EXISTS mv_products_orders CASCADE;

The above command drops both materialized views: `mv_products_orders`, as well as `my_mv` which was created based on it.

## Conclusion

Like all optimization tools, the use of materialized views involves trade-offs. It is important to understand the specific needs of each use-case before deciding whether a tool is the right fit for it.

### Benefits

* Like views, materialized views offer a consistent interface to the database. Materialized views abstract away the database design and implementation details to present a consistent querying interface to the API layer.

* Materialized views cache the results of the query in a persistent structure so it can be accessed without having to be recomputed. This saves time on repeatedly accessed complex queries.

### Costs

* Materialized views consume additional storage but in practice, the cost of extra storage is not a deciding factor when storage is cheap.

* Data recency: if the underlying tables are frequently updated, it is likely that the data cached by the materialized view will have been partially outdated by the time it is used. This is a problem for queries which need to return real-time data. Automated refreshes help with data recency, but their use comes with trade-offs, especially for large tables and for write-heavy databases.

### Use-cases

Materialized views help improve performance, often significantly, in situations where the system needs to handle high volumes of the same (known in advance) complex queries. For example:

* reporting and analytics applications which involve complex queries on large tables

* database designs involving unstructured or semi-structured data, where querying is inefficient

* dashboards presenting collated and/or consolidated (daily, monthly, and so on) information

* queries involving external tables and data-stores - where it can be slow and/or expensive to repeatedly query the data source

* providing API services to third parties where contractual requirements often necessitate a consistent API structure, and where heavy loads are expected

The use of materialized views is not necessary when the underlying query is simple and/or fast.

Materialized views are **not** suitable for:

* queries that power real-time applications like live trading, online bidding, sports scores, messaging, live news feeds, and so on.
