# Server requirements

ColdIgniter is written for ColdFusion MX 6.1. A default install is enough: no setting in the ColdFusion Administrator has to be changed to run it.

## ColdFusion

| | |
|---|---|
| Server | ColdFusion MX 6.1 |
| Java | 1.4.2, the version that ships with MX 6.1 |

ColdFusion MX 6.0 is the oldest version that has components (`cfcomponent`, `cffunction`, `cfargument`). MX 6.1 added `super` and ships with Java 1.4.2, which has `java.util.LinkedHashMap` and `java.util.regex`. The port uses both in many places.

## Database

The database driver is `SQLSRV`, for Microsoft SQL Server 2000 SP3.

ColdIgniter connects only through a data source of the ColdFusion Administrator. The name of the data source goes in the `DSN` setting of the database group. The login is kept in the data source, not in the application.

## Encryption

The default cipher is AES with a 128-bit key. This is what a default Java 1.4.2 install allows.

AES with a 256-bit key is possible when the administrator installs the JCE unlimited strength policy files. It is never required.

## What ColdIgniter keeps in memory

ColdIgniter does not use the `server` scope. It names one application per site and keeps a few tables there: class names, paths and rules. These tables hold no secret and no value of the `.env` file.
