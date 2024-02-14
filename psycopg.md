# How to Access PostgreSQL from Python Using Psycopg

## Introduction

To connect to a PostgreSQL database from Python, you can use:

1. An Object-Relational Mapping (ORM) tool, [like SQLAlchemy](https://www.vultr.com/docs/how-to-use-sql-databases-in-python-with-sqlalchemy/), or
1. A driver, like Psycopg, to directly issue raw SQL queries to the database

Using an ORM is the right choice when:

* queries are not expected to be complex
* when application developers also work on the database - the database is tightly coupled to the application code
* the team is not experienced in SQL

Using a driver instead of an ORM is the right choice when:

* the nature of the application makes it necessary to write (and optimize) complex queries
* parts of the application might need to be ported to a different programming language or framework
* the team is already proficient in SQL

### Prerequisites

To benefit from this guide, you need some familiarity with both Python and PostgreSQL. It is assumed you have [PostgreSQL running either on a standalone server](https://www.vultr.com/docs/how-to-install-configure-backup-and-restore-postgresql-on-ubuntu-20-04-lts/) or as an [instance of Vultr Managed Databases for PostgreSQL](https://www.vultr.com/docs/postgresql-managed-database-guide/). It is also assumed you are comfortable executing raw SQL commands.

### Compatibility

The code examples in this guide are based on Python 3.9.15, Psycopg 3.1.5, and PostgreSQL 14.5. They should be compatible with all recent versions of Python, PostgreSQL, and Psycopg3. The installation commands are based on Debian and `apt`. Use the appropriate package manager for your Operating System. Python commands are identified with the `>>>` prompt and SQL commands are identified with the `=#` prompt.

#### Psycopg2

Although (the first stable version of) Psycopg3 was [released in October 2021](https://www.postgresql.org/about/news/psycopg-30-released-2328/), many applications continue to use Psycopg2. It is recommended that new applications are built with Psycopg3. In general, [most commands should be cross-compatible](https://www.psycopg.org/articles/2020/11/24/psycopg3-adaptation/) but there are [a few differences between Psycopg2 and Psycopg3](https://www.psycopg.org/psycopg3/docs/basic/from_pg2.html). Where relevant, instructions for Psycopg2 are also mentioned.

## How to Install

If you are starting from a fresh installation, install Python3 and `pip`:

    # apt install python3 pip

Psycopg depends on the *libpq* library. Install it on the Operating System:

    # apt install libpq-dev

Note that on RHEL and derivative systems, the name of the *libpq* package is `libpq-devel`.

Install Psycopg3 using the PyPI installer:

    $ pip install --upgrade pip ; pip install psycopg

Note that the package name `psycopg` refers to the Psycopg3 package. The package name for Psycopg2 is `psycopg2`.

## Connect to the Database

In a new Python terminal, import the Psycopg module:

    >>> import psycopg

### Connect to Local Database Instance

The `connect()` method connects to a new database session. Connect to an instance of PostgreSQL running locally:

    >>> conn = psycopg.connect(dbname="postgres", user="postgres") 

If needed, specify the database name, username, password, host address, and port number using the parameters `dbname`, `user`, `password`, `host` and, `port`, respectively:

    >>> conn = psycopg.connect(dbname="MY_DATABASE", user="MY_USERNAME", password="MY_PASSPHRASE", host="IP_ADDRESS", port="PORT" ) 

### Connect to an Instance of Vultr Managed Databases for PostgreSQL

When you create a new instance on Vultr Managed Databases for PostgreSQL, it automatically makes a default database `defaultdb` with the default username `vultradmin`. Using these, the connection command looks like this:

    >>> conn = psycopg.connect(dbname="defaultdb", user="vultradmin", password="MY_PASSPHRASE", host="vultr-prod-complete-host-name-string.vultrdb.com", port="PORT") 

Copy the relevant parameters from the *Connection Details* page of the Vultr Managed Databases for PostgreSQL instance.

### Create Cursor

In standard database terminology, a cursor is a pointer to an individual row in a result set consisting of many rows. Psycopg cursors extend this concept. Psycopg cursors can send queries to the database and also access the results. To create a new cursor, use the `cursor()` method of the `connection` object:

    >>> cur = conn.cursor()

This creates an instance of the `cursor` class. To send queries to the database, use the `execute()` method of the `cursor` class.

## `SELECT` Queries

PostgreSQL has an inbuilt view named `pg_timezone_names` - it contains the complete list of names of timezones allowed in PostgreSQL. Run a `SELECT` query on this view:

    >>> cur.execute("SELECT * FROM pg_timezone_names")

The result, however, is not directly displayed on the screen. The `execute` command succeeds silently - if the query ran successfully, the result is stored in the cursor and you are back at the Python prompt. The results of the query are stored in array form, each row corresponding to an array element. If the query failed, it throws an error at the prompt. To view the results of the query, use the `fetchone()`, `fetchmany()`, or `fetchall()` methods on the cursor object.

Before viewing the results, check some relevant details of the cursor object.

The size (length) of the array is the number of rows returned:

    >>> cur.rowcount

To get the row number where the cursor is currently at:

    >>> cur.rownumber

This is the number of the row that will be fetched next. Since no rows have been fetched as of yet, the cursor should be at row number 0.

### Fetch Results One Row at a Time

Fetch the first row returned in the cursor:

    >>> cur.fetchone()

Check again the row number where the cursor is currently at:

    >>> cur.rownumber

It should have moved forward by 1.

### Fetch Results N Rows at a Time

When the result set contains a large number of rows, it can be advantageous to fetch and process a few rows at a time. The `fetchmany()` function does this:

    >>> result2 = cur.fetchmany(2)

This returns an array of (the next) 2 rows of the result:

    >>> result2

Use standard array syntax, like `result2[1]` to check the contents of `result2`.

`fetchmany(N)` moves the cursor ahead by N rows.

### Fetch the Entire Result at Once

It is also possible to fetch all the rows in the result set at once:

    >>> results = cur.fetchall()

This returns an array, with each row of the result corresponding to an element in the array. Check again the row number where the cursor is currently at:

    >>> cur.rownumber

It should have moved to the end of the array. Trying to fetch results again from the cursor will return either `NULL` or an empty array.

### Practical Tip

Even when the cursor fetches 1 or N rows at a time, the entire result set is still loaded into the client's memory.

So, when the result contains a large number (millions) of rows, fetching and loading them can need more memory than the client has available. In such situations, the preferred approach is to [paginate the results](https://www.vultr.com/docs/how-to-page-postgresql-data-with-php/). The database sends only a certain number of rows per query. Pagination of query results is beyond the scope of this guide.

## Table Modification Queries

Create a table, `test_table`, with a few columns of different data types - text (`varchar`), `integer`, `float`, `date`, `timestamp`, `timestamp with timezone`, `time`, `json`, and `jsonb`.

    >>> cur.execute("CREATE TABLE test_table (my_id SERIAL PRIMARY KEY, my_name VARCHAR(20), my_number INTEGER, my_float FLOAT, my_date DATE, my_timestamp TIMESTAMP, my_timestamp_tz TIMESTAMPTZ, my_time TIME, my_json JSON, my_jsonb JSONB )")

Similarly, you can add columns to the table:

    >>> cur.execute("ALTER TABLE test_table ADD COLUMN new_name VARCHAR(100)")

To drop a column:

    >>> cur.execute("ALTER TABLE test_table DROP COLUMN new_name ")

## Data Modification Queries

Before adding data to the table, declare Python variables to hold the values of data fields:

    >>> py_name = "first'last%s" ; py_number = 1234 ; py_float = 3.14

Notice that the `py_name` variable contains some special characters. In practice, the values of the data fields might come, for instance, from the front end of a web application, or they may be the result of other computations.

**Practical Tip:** SQL injection is a serious security issue. It is important to sanitize all data coming from users. The right way to do it is to first extract relevant data fields, sanitize them, and then pass them to the database.

### Parametrized Queries

Use the parametrized form of the `execute()` function to insert values into `test_table`:

    >>> cur.execute("INSERT INTO test_table (my_name, my_number, my_float) VALUES (%s, %s, %s)", [py_name, py_number, py_float])

The first argument to the `execute()` function is the query itself enclosed in double quotes. In the query, `%s` is a placeholder for each parameter. The second argument is an array containing the list of values of the parameters.

Values for the parameters in the query are taken sequentially from the array passed in the second argument. The first `%s` corresponds to the first item in the array, `py_name`. The second `%s` corresponds to the second item in the array, `py_number`.

In principle, it is possible to directly pass the values of the parameters:

    >>> cur.execute("INSERT INTO test_table (my_name, my_number) VALUES (%s, %s)", ["first last", 1234])

In practice, it is recommended to **not** do this. Hard coding the parameter values in the query can lead to complications when the values contain special characters.

### Named Parameters

When a query contains a large number of parameters, it is difficult to keep track of the order of the parameters and the values in the array. Named parameters solve this problem. Name the placeholders and the list of parameters:

    >>> cur.execute("INSERT INTO test_table (my_name, my_number) VALUES (%(param_name)s, %(param_number)s)", {'param_number': py_number, 'param_name': py_name})

As shown above, when using named parameters, it is no longer necessary to keep track of the order in which they are written.

### `INSERT...RETURNING`

The `INSERT`...`RETURNING` syntax of PostgreSQL is helpful to return values from an `INSERT` query. This is necessary, for example, when inserting values into tables that have an auto-incremented ID field.

    >>> cur.execute("INSERT INTO test_table (my_name) VALUES (%s) RETURNING my_id", [py_name])

The above command inserts a new row. And it returns the auto-incremented ID field, `my_id`, of the newly inserted row. Use the `fetchone()` method on the cursor object, `cur`, to get the value of `my_id` for the inserted row:

    >>> cur.fetchone()

## Commit and Roll Back Transactions

At this point, a table has been created and values inserted into it. Check the inserted values (from within the same session):

    >>> cur.execute("SELECT * FROM test_table")

Fetch the result set:

    >>> cur.fetchall()

This shows all the data in the table. Now check the values of `test_table` from the PostgreSQL command line:

    =# SELECT * from test_table ;

This returns an error saying the table doesn't exist. The PostgreSQL command line is a different session than the one created by Psycopg. That session is unable to see the changes from the Psycopg session because the transaction is not yet committed. Similarly, if you create a new connection using another Python command line, and use it to run a query on `test_table`, it will also return an error. The new changes are only visible within the session that created them.

To make changes permanent (and viewable by all), commit all outstanding transactions using the `commit()` method of the `connection` object:

    >>> conn.commit()

Check again the values in the PostgreSQL command line:

    =# SELECT * from test_table ;

The changes should now be visible at the PostgreSQL command line. Similarly, they will also be visible from other Psycopg connections.

### Roll Back Transactions

Rolling back a transaction refers to undoing the uncommitted changes. In the earlier examples, if you had rolled back the transaction instead of committing it, all changes would have been lost. Add a new row to the table:

    >>> cur.execute("INSERT INTO test_table (my_name ) VALUES (%s)", ["new name"] )

Check that this value has been inserted into the table:

    >>> cur.execute("SELECT * FROM test_table WHERE my_name = (%s)", ["new name"] ) ; cur.fetchone()

Now roll back the transaction instead of committing it:

    >>> conn.rollback()

Query the table again for the newly inserted value:

    >>> cur.execute("SELECT * FROM test_table WHERE my_name = (%s)", ["new name"] )

Check the result set:

    >>> cur.fetchone()

It does not return any value. The changes have been undone.

### Aborted Transactions

Consider a situation when you inadvertently enter an erroneous query.

    >>> cur.execute("SELECT * FROM test_table1")

`test_table1` does not exist, and the command returns an error. Now fix the mistake and enter the correct query:

    >>> cur.execute("SELECT * FROM test_table")

This returns another error:

    psycopg.errors.InFailedSqlTransaction: current transaction is aborted, commands ignored until end of transaction block

Psycopg starts a new transaction for every query, including simple `SELECT`s. A failed query leads to a failed transaction. The cursor and the connection will not accept new queries until the failed transaction is rolled back.

    >>> conn.rollback()

You can now enter the correct query and get results. In principle, it is also possible to commit the transaction and free up the connection. But it is inadvisable to commit transactions if there were errors.

## Managing Transactions

It is advisable to manually close the connections and cursors when they are no longer needed. In [Psycopg3, transaction management is baked into the Python block context model](https://www.psycopg.org/psycopg3/docs/basic/transactions.html). This gives better control over managing transactions. Open a connection (or create a cursor) using the Python `with` statement at the start of a block:

    # pseudo-code
    with psycopg.connect() as conn:
        cur = conn.cursor()
        cur.execute(......)

    # exit the block 

When it exits the block, it automatically commits the transaction if there were no errors and closes the connection. It is possible to configure the settings so it rolls back the transaction in case a query fails in the block.

[It is also possible to manage transactions in Psycopg2](https://www.vultr.com/docs/how-to-implement-postgresql-database-transactions-with-python-on-ubuntu-20-04/), but it needs more boilerplate code compared to Psycopg3. A detailed discussion of Transactions is beyond the scope of this guide.

## Common Data Types

In general, most data types are automatically handled by the Psycopg adapter. This includes strings, integers, floating point numbers, etc. In some cases, it is necessary to pre-process the data before writing it to the database. This section introduces two commonly used data types - JSON  and timestamps, where some pre-processing is needed.

### Date and Time

To use dates and times in Python, use the `datetime` module:

    >>> import datetime

Insert the current timestamp into a column of type `timestamp` and commit the transaction:

    >>> cur.execute("INSERT INTO test_table (my_timestamp) VALUES (%s) ", [datetime.datetime.now()]) ; conn.commit()

The same syntax also works for timestamps with time zone:

    >>> cur.execute("INSERT INTO test_table (my_timestamp_tz) VALUES (%s) ", [datetime.datetime.now()]) ; conn.commit()

To change the time zone, run the `SET TIME ZONE` query on the database:

    >>> cur.execute("SET TIME ZONE 'Europe/Rome'") ; conn.commit()

The above command sets the time zone of the database to `Europe/Rome`. All further values inserted into a column of type `timestamp with time zone` will bear this time zone. The complete list of timezones in PostgreSQL can be seen in the view `pg_timezone_names`.

You can insert an arbitrary date, time, or datetime. Create a date variable using the `date()` function with the arguments *(year, month, date)*:

    >>> py_date = datetime.date(2022, 12, 25)

Insert it into a column of type `date`:

    >>> cur.execute("INSERT INTO test_table (my_date) VALUES (%s) ", [py_date]) ; conn.commit()

Create a time variable using the `time()` function with the arguments *(hours, minutes, seconds, microseconds)*:

    >>> py_time = datetime.time(23, 22, 21, 20)

Insert it into a column of type `time`:

    >>> cur.execute("INSERT INTO test_table (my_time) VALUES (%s) ", [py_time]) ; conn.commit()

Create a datetime variable using the `datetime()` function with the arguments *(year, month, day, hours, minutes, seconds, microseconds)*:

    >>> py_datetime = datetime.datetime(2022, 12, 25, 23, 22, 21, 20)

Insert it into a column of type `timestamp`:

    >>> cur.execute("INSERT INTO test_table (my_timestamp) VALUES (%s) ", [py_datetime]) ; conn.commit()

Check the inserted values using either the PostgreSQL command line or in Python, as shown in earlier examples.

### JSON Data

To handle JSON data in Python, import the `json` module:

    >>> import json

To handle JSON data in Psycopg, you need the Jsonb function from the `psycopg.types.json` module of Psycopg:

    >>> from psycopg.types.json import Jsonb

Note: [Json works slightly differently in Psycopg2 - the documentation is a good starting point to learn more about it](https://www.psycopg.org/docs/extras.html?highlight=json#json-adaptation).

#### Handling JSON Data

The Python data type `dictionary` consists of key value pairs. Declare a Python dictionary variable, containing keys with values of different data types:

    >>> py_dict1 = {"key1": ["foo", None, 1.0, 2], "key2": "bar", "key3": 20 }

Convert the dictionary object to a JSON variable:

    >>> py_json1 = Jsonb(py_dict1)

#### `INSERT` JSON Data

As before, insert the Python variable containing the JSON object into the database:

    >>> cur.execute("INSERT INTO test_table (my_json) VALUES (%s)", [py_json1]) ; conn.commit()

JSONB data is inserted the same way:

    >>> cur.execute("INSERT INTO test_table (my_jsonb) VALUES (%s)", [py_json1]) ; conn.commit()

#### `SELECT` JSON Data

Query the table for the inserted JSON data:

    >>> cur.execute("SELECT my_json FROM test_table WHERE my_json IS NOT NULL")

Fetch the first row of the result set:

    >>> json1 = cur.fetchone() 

The above command returns an array. Get the first (and only) element of that array:

    >>> json1 = json1 [0]  

`json1` is of type dictionary. To work further on it, use the standard Python dictionary functions like `keys`, `get`, and so on.

    >>> json1.get("key1")

The above command returns the value corresponding to the key `key1`. JSONB also works the same way.

## Conclusion

This guide introduced the basic concepts of directly connecting to a PostgreSQL from Python, using Psycopg. To use it in practice, learn about [handling other data types](https://www.psycopg.org/psycopg3/docs/basic/adapt.html), [transaction management](https://www.psycopg.org/psycopg3/docs/basic/transactions.html), and [connection pooling](https://www.psycopg.org/psycopg3/docs/advanced/pool.html).
