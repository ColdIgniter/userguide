# Server Requirements

ColdIgniter is written for ColdFusion MX 6.1. A default install is enough: apart from the data source of your database, no setting in the ColdFusion Administrator has to be changed to run it.

## ColdFusion and Required Extensions

| | |
|---|---|
| Server | ColdFusion MX 6.1 |
| Java | 1.4.2, the version that ships with MX 6.1 |

ColdFusion MX 6.0 is the oldest version that has components (`cfcomponent`, `cffunction`, `cfargument`). MX 6.1 added `super` and ships with Java 1.4.2, which has `java.util.LinkedHashMap` and `java.util.regex`. The port uses both in many places. On a server older than 6.1 the front controller stops with a 503 message that names the version it found.

No add-on, extension or custom tag is required.

## Optional Extensions

ColdIgniter needs no optional extension. These parts of CodeIgniter 4 are not ported yet, so there is nothing to install for them: the CURLRequest class, the Image class (with its GD and ImageMagick handlers) and the Memcached and Redis cache handlers.

## Supported Databases

A database is required for most web application programming. The supported database is Microsoft SQL Server 2000 SP3, with the `SQLSRV` driver.

ColdIgniter connects only through a data source of the ColdFusion Administrator. In the database group of `app/Config/Database.cfc`, set two values:

| Setting | Value |
|---|---|
| `DBDriver` | `SQLSRV`. The file ships with `MySQLi`, as in CodeIgniter, so you must change it. |
| `DSN` | The name of the data source. |

The login is kept in the data source, not in the application.

!!! note "Difference from CodeIgniter"
    CodeIgniter 4 has drivers for MySQL, PostgreSQL, SQLite3, SQL Server and Oracle. ColdIgniter has the SQL Server driver only.

## Encryption

The default cipher is AES with a 128-bit key. This is what a default Java 1.4.2 install allows.

AES with a 256-bit key is possible when the administrator installs the JCE unlimited strength policy files. It is never required.

## What ColdIgniter keeps in memory

ColdIgniter stores nothing in the `server` scope, which every site of a ColdFusion server can read. It uses one application scope per site and keeps these things there:

- tables of the framework: class names, paths and rules;
- with the file cache handler, copies of the cached values that were read last. The default is up to 100 for a site (`memoryEntries` in `app/Config/Cache.cfc`; 0 turns this off);
- cached pages, when a controller uses page caching and the site is not in development mode.

The tables of the framework hold no secret and no value of the `.env` file. Cached values and cached pages hold what your application puts in the cache.
