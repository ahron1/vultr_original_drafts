# How to Calculate Distances Using PostgreSQL

## Introduction

Many modern applications need to calculate distances - between two people, between a business and a person, and so on. Locations are typically represented as latitude-longitude pairs (lat/long coordinates). To do calculations involving distances, these are some of the approaches you can take (and their consequences):

* Get the location coordinates from the database, and use an application layer function to compute the distance. This approach is easy to set up, but makes it difficult to run many kinds of distance-based queries. Consider an example where you want to find restaurants within a 5-kilometer radius of an individual's current position. To do this, the application fetches the list of all restaurants and then finds the ones within the radius.
* Write a PL/SQL function to compute the distance within the database. Because the data is processed at the database layer, this approach is operationally efficient. But setting up a PL/SQL function is time-consuming and error-prone. The next two approaches offer the same level of efficiency at a lower setup cost.
* Use a module like PostGIS. PostGIS is a fully featured Geographic Information Systems (GIS) module. Its use is overkill when you only need to calculate distances.
* Use the `earthdistance` module built into PostgreSQL. This module provides an accessible approach to calculating distances, and it suffices for most use cases. It also makes it possible to write distance-based queries.

## Prerequisites

This is a hands-on guide. To benefit from it, you are expected to have prior practical experience with PostgreSQL. You are also expected to already have PostgreSQL running either on a [standalone server](https://www.vultr.com/docs/how-to-install-configure-backup-and-restore-postgresql-on-ubuntu-20-04-lts/) or [as an instance on Vultr Managed Databases for PostgreSQL](https://www.vultr.com/docs/postgresql-managed-database-guide/).

It is assumed you have access to the PostgreSQL command line, and you are comfortable issuing SQL commands. It is helpful to know [how Common Table Expressions (CTEs) work in PostgreSQL](https://www.postgresql.org/docs/current/queries-with.html) - the examples use CTEs.

Extensions are available only in the database where they are loaded. Before starting, check that you are connected to the database in which you want to use the module. To connect to a specific database, use:

    =# \c DATABASE_NAME

Note that `=#` refers to the PostgreSQL `psql` prompt.

By default, a standalone PostgreSQL installation connects to the database `postgres` as the user `postgres`. On Vultr Managed Databases for PostgreSQL, the default database is `defaultdb` and the default user is `vultradmin`. Following this guide without explicitly connecting to a specific database will load the extension(s) into the default database. This is not a problem, only something to be aware of.

### Compatibility

The instructions in this guide are tested on PostgreSQL version 14.5 running on Ubuntu 22.04 and Vultr Managed Databases for PostgreSQL. They should be compatible with all recent versions of PostgreSQL on all recent Operating Systems.

## How to Install

The `earthdistance` module needs the `cube` module as a dependency. `cube` is also a builtin extension, you do not need to install it. Load the `cube` module as an extension:

    =# CREATE EXTENSION cube ; 

Now load the `earthdistance` extension:

    =# CREATE EXTENSION earthdistance ;

Check that both extensions are loaded and ready to use:

    =# SELECT * FROM pg_extension ;

The list of extensions should include `cube` and `earthdistance`.

## Create Test Data

### Get Location Data

A typical consumer-facing web or mobile application gets the location data of the user in different ways:

* The device, via its GPS sensors, gets the lat/long coordinates. A service like the Location API from Google Play Services or Apple's Core Location API handles this within the app. The app then shares the coordinates with the server.
* The user enters an address. A service like Google's Geocoding API takes in the address and outputs its geographical coordinates.
* The user application uses a service like Google's Geolocation API to get the coordinates.

For the examples in this section, get the locations using either Google Maps or a service like [Itilog](https://www.itilog.com/). On Google Maps, right-click on any location to see its coordinates (latitude and longitude) in the pop-up context menu. Left-click on the shown coordinates to copy them to the clipboard. On Itilog, you can type out a specific address or click on a location on the map to see its coordinates in the box above. Itilog is based on Google APIs.

### Create Table

Create a table to store information about places:

    CREATE TABLE places (id SERIAL PRIMARY KEY, city VARCHAR (20) UNIQUE, lat FLOAT8, long FLOAT8, UNIQUE (lat, long)) ;

The table has 4 columns - an automatically generated ID field, the name of a city, and its lat/long coordinates. The uniqueness constraints in the query above are not related to the `earthdistance` module. They are imposed only to sanitize the data in a production database. You can skip the uniqueness constraints for testing.

Create a table with information about people:

    CREATE TABLE people (id SERIAL PRIMARY KEY, name VARCHAR (20), lat FLOAT8, long FLOAT8 ) ;

Having two tables allows you to test how to make distance-based queries across different tables.

### Insert Test Data

Insert a few rows with the names of places and their lat/long coordinates:

    INSERT INTO places (city, lat, long) VALUES ('London', 51.5073219, -0.1276474), ('Edinburgh', 55.9533456, -3.1883749), ('Paris', 48.8588897, 2.320041), ('Amsterdam', 52.3727598, 4.8936041), ('Washington', 38.8950368, -77.0365427), ('Delhi', '28.6138954', '77.2090057'), ('Sydney', -33.8698439, 151.2082848), ('Null Island', 0, 0), ('North Pole', 90, 0), ('South Pole', -90, 0) ;

Insert a few rows in the people table, corresponding to people living in different places:

    INSERT INTO people (name, lat, long) VALUES ('Londoner', 51.5073219, -0.1276474), ('Edinburgher', 55.9533456, -3.1883749), ('Parisian', 48.8588897, 2.320041), ('Amsterdammer', 52.3727598, 4.8936041), ('Washingtonian', 38.8950368, -77.0365427), ('Delhite', '28.6138954', '77.2090057'), ('Sydneysider', -33.8698439, 151.2082848) ;

## How It Works

### The `earthdistance` module

The `earthdistance` module is packaged as a PostgreSQL extension. It assumes the earth to be a perfect sphere. It represents points on the globe in a three-dimensional grid system. The origin of this 3D grid is at the center of the earth. Each point on the 3D grid is represented as a set of (x, y, z) coordinates. The X-Y plane contains the equator, and the Z-axis joins the poles. This 3D grid system is handled by the `cube` module.

Distance between points is measured along the surface of the earth - this is the *great circle distance*. Geographically, locations are represented by latitude and longitude pairs. Latitudes and longitudes are stored as double-precision floating point numbers (`float8`).

### Coordinates to Cubes

Points on the earth are naturally represented as lat/long coordinates. But the `earthdistance` module represents points in a 3D grid. The distance-calculating functions of the module are also based on points in a 3D system. To convert the location of a point from the lat/long coordinate system to the 3D grid system, use the `ll_to_earth` function.

    SELECT 
    city, lat, long, 
        ll_to_earth(lat, long) AS earth_xyz
    FROM places ;

The above query shows the latitude and longitude of each place, as well as its location in the 3D grid system (as computed by the `ll_to_earth` function). The 3D grid location is represented by a set of (x, y, z) numbers. Observe the 3D grid coordinates of [Null Island](https://en.wikipedia.org/wiki/Null_Island), the North Pole, and the South Pole. They correspond to (R, 0, 0), (0, 0, R), and (0, 0, -R), respectively, where *R* is the radius of the earth. Note that a zero coordinate in the 3D grid might not be represented as exactly `0` but as a very small number of the order of 10 to the power of -10.

#### Caveat

In reality, the earth is not a perfect sphere. It is slightly flattened at the poles, and it bulges out at the equator. The `earthdistance` module neglects this and assumes the earth to be a perfect sphere. If your application needs highly accurate distances that take into account the earth's actual curvature (for example, if you want to calculate airline routes over the poles), use the PostGIS module. For most casual applications, the builtin `earthdistance` module is sufficient.

Use the `earth()` function to see the radius of the earth (as per the module's models) in meters:

    SELECT earth();

## Distance-based Queries on the Same Table

Distances are computed using the `earth_distance` function. This function takes in as input two points (represented in the 3D grid system) on the surface of the earth. Therefore, the lat/long coordinates are first converted to (x, y, z) coordinates using the `ll_to_earth` function.

### Distance between Two Places

In the following CTE, the first and second sub-queries get the details (lat/long coordinates and name) of two cities.

    WITH 
    place1 AS (
        SELECT lat, long, city
        FROM places 
        WHERE city = 'Delhi'
    ),
    place2 AS (
        SELECT lat, long, city
        FROM places
        WHERE city = 'Paris'
    )
    SELECT
        place1.city AS place1,
        place2.city AS place2,

        earth_distance(
            ll_to_earth(place1.lat, place1.long),
            ll_to_earth(place2.lat, place2.long)
        ) AS distance_meters

    FROM place1, place2 ;

The above query returns the distance between Delhi and Paris in meters. Notice that the calculated distance is a floating point number.

#### Conversion of Units

Round off the distance by casting it to the `INTEGER` data type. Convert it to kilometers by dividing it by 1000. To convert to miles, divide by 1609.344.

    WITH 
    place1 AS (
        SELECT lat, long, city
        FROM places 
        WHERE city = 'Delhi'
    ),
    place2 AS (
        SELECT lat, long, city
        FROM places
        WHERE city = 'Paris'
    )
    SELECT
        place1.city AS place1,
        place2.city AS place2,

        (earth_distance(
            ll_to_earth(place1.lat, place1.long),
            ll_to_earth(place2.lat, place2.long)
        -- typecasting and converting to miles
        )/1609.344)::integer AS distance_miles

    FROM place1, place2 ;

The above query shows the Delhi-Paris distance in miles. Pay attention to the order of operations and the parentheses - divide the distance in meters by 1609.344, then cast the result to `INTEGER`.

### Places within a Radius

To find all places within a certain radius of another place, add a distance-based `WHERE` condition to the `SELECT` clause:

    WITH 
    place1 AS (
        SELECT lat, long, city
        FROM places 
        WHERE city = 'London'
    ),
    place2 AS (
        SELECT lat, long, city
        FROM places
        WHERE city <> 'London'
    )
    SELECT
        place1.city AS place1,
        place2.city AS place2,

        earth_distance(
            ll_to_earth(place1.lat, place1.long),
            ll_to_earth(place2.lat, place2.long)
        )::integer/1000 AS distance_kilometers

    FROM place1, place2
    WHERE
        earth_distance(
            ll_to_earth(place1.lat, place1.long),
            ll_to_earth(place2.lat, place2.long)
        )/1000 < 1000
    ORDER BY distance_kilometers ;

This query finds all places within 1000 kilometers of London. Note the inequality condition in the second sub-query.

## Distance-based Queries on Different Tables

The last section showed how to run distance-based queries on the same table. In many cases, you need to run the query across multiple tables. For example, if you have one table for places and another for people, and you want to find all people within some radius of a place.

### All People within a Radius

The overall structure of this query is similar to earlier ones. One of the sub-queries is used to fetch details from the `people` table. The other sub-query gets information from the `places` table. The query below finds all people within a 1000-kilometer radius of London.

    WITH
    place AS (
        SELECT lat, long, city
        FROM places 
        WHERE city = 'London'
    ),
    person AS (
        SELECT lat, long, name
        FROM people
    )
    SELECT
        place.city AS place,
        person.name AS person,
        earth_distance(
            ll_to_earth(place.lat, place.long),
            ll_to_earth(person.lat, person.long)
        )::integer/1000 AS distance_kilometers

    FROM place, person
    WHERE 
        earth_distance(
            ll_to_earth(place.lat, place.long),
            ll_to_earth(person.lat, person.long)
        )::integer/1000 < 1000
    ORDER BY distance_kilometers ;

## Conclusion

This article discusses how to use the `earthdistance` module of PostgreSQL to calculate distances. The [official documentation](https://www.postgresql.org/docs/current/earthdistance.html) has more information about the module and some additional functions. For more advanced geographical functions, use the [PostGIS module](https://postgis.net/). In practice, you parametrize the queries to receive inputs from the application programming language. For example, when using Python, you might use either an ORM like [SQLAlchemy](https://www.vultr.com/docs/how-to-use-sql-databases-in-python-with-sqlalchemy/) or a driver like [Psycopg](https://www.psycopg.org/psycopg3/docs/) to interface with the database.

    # Notes to the editor
    The second paragraph in the Prerequisites section has a link to the PostgreSQL documentation of CTEs. After the Vultr article on CTEs is published, the link should be updated with that article. The last paragraph has a link to the Psycopg website. After the Vultr article on Psycopg is published, the link should be updated with that article. 
