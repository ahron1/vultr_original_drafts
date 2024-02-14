# How to Install the PostGIS Extension for PostgreSQL

## Introduction

PostGIS is a full-fledged Geographic Information System (GIS) data modeling tool. It has many GIS-specific data types and functions. PostGIS is available as an extension for the PostgreSQL database, it is not a standalone software.

This guide shows how to install PostGIS on:

1. PostgreSQL running on a Ubuntu server / VM.
1. Vultr Managed Databases for PostgreSQL.

### Prerequisites

Before starting with this guide, you are expected to have either:

* [PostgreSQL installed on a standalone server / VPS running Ubuntu](https://www.vultr.com/docs/how-to-install-configure-backup-and-restore-postgresql-on-ubuntu-20-04-lts/) or,
* [an instance on Vultr Managed Databases for PostgreSQL](https://www.vultr.com/docs/postgresql-managed-database-guide/).

It is assumed you have access to the PostgreSQL command line and you are comfortable issuing basic SQL commands. PostGIS is available only in the database where it has been loaded. Before loading the PostGIS extension, ensure that you are connected to the database in which you want to use PostGIS. To connect to a specific database use:

    =# \c DATABSE_NAME

By default, a standalone PostgreSQL installation connects to the `postgres` database as the user `postgres`. An instance of Vultr Managed Databases for PostgreSQL connects to the `defaultdb` database as the user `vultradmin`. Following this guide without explicitly connecting to a specific database will load PostGIS into the default database.

Note that `=#` refers to the PostgreSQL `psql` prompt, `#` refers to the root prompt on the Operating System, and `$` refers to a regular user prompt.

### Compatibility

The instructions in this guide are based on PostgreSQL version 14 and PostGIS 3.2 running on Ubuntu 22.04 and Vultr Managed Databases for PostgreSQL. They should be compatible with all recent versions of the software and Operating System.

Note that in the case of standalone servers, you need to have only one version of PostgreSQL installed. Handling two or more versions installed on the same system is beyond the scope of this guide.

## How to Install

Assuming that PostgreSQL is already available, PostGIS installation consists of 2 steps:

1. Install `postgis`, the PostGIS package, from the Operating System's package manager
1. Load PostGIS into a PostgreSQL database

While this guide demonstrates the first step for Ubuntu, the general procedure is the same for all Operating Systems. The second step is the same for both standalone servers as well as Vultr Managed Databases for PostgreSQL.

## Installation Step 1 - Install Operating System Packages for PostGIS

### Install on Ubuntu

#### Optional - Check Compatibility

In general, there should be no compatibility issues on Ubuntu. Check the version of PostgreSQL installed on your Operating System. On the PostgreSQL command line, enter:

    =# SELECT version();

Check the version of PostGIS available on the `apt` package manager:

    $ apt search ^postgis$

Check the [compatibility page](https://trac.osgeo.org/postgis/wiki/UsersWikiPostgreSQLPostGIS) to ensure that the available version of PostGIS is compatible with the installed version of PostgreSQL.

#### Install the Package

Install the `postgis` package on Ubuntu using the `apt` package manager.

    # apt install postgis

This will install the latest version of PostGIS available for the Operating System.

### Install on Vultr Managed Databases for PostgreSQL

On Vultr Managed Databases for PostgreSQL, the PostGIS extension package is already installed. You only need to load it (as shown in the next step).

## Installation Step 2 - Load the PostGIS Extension in PostgreSQL

The second step of the installation is the same across both standalone servers and managed databases. Log in to the PostgreSQL command line using the *psql* tool, *pgAdmin*, or another front-end tool that allows you to issue queries to the PostgreSQL server. For Vultr Managed Databases for PostgreSQL, find the credentials from the Connection Details page of your instance.

### Check Availability

Check if the extension is available to be loaded:

    =# SELECT * FROM pg_available_extensions;

After installing PostGIS on the Operating System, it should be included in this list. 

### Load the Extension

Use the `CREATE EXTENSION` command of PostgreSQL to create a new extension for PostGIS:

    =# CREATE EXTENSION postgis ;

The PostGIS extension should now be loaded into the database.

### Check Installation

Check that the PostGIS extensions is loaded:

    =# SELECT * FROM pg_extension ;

The above command shows the list of all loaded extensions. You can also check the version of PostGIS installed:

    =# SELECT postgis_version();

## Conclusion

PostGIS has now been loaded into your PostgreSQL database. [The PostGIS documentation](https://postgis.net/documentation/) is a good starting point to learn more about how to handle GIS data in PostgreSQL.
