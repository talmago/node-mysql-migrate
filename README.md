# node-mysql-migrate

## Overview
Simple migration tool for Mysql/Node.js

# Quick Start

#### Install

```sh
npm install -g https://github.com/talmago/node-mysql-migrate
```

After the installation, you can start using the command line.


#### Command line

```sh

$ mysql-migrate --help

  Usage: node-mysql-migrate [options] [command]


  Commands:

    info                show revision information
    clean               drops all objects in the managed schema
    repair              repair migration failures
    baseline <version>  baseline existing schema to initial version
    migrate [version]   migrate schema to new version

  Options:

    -h, --help     output usage information
    -V, --version  output the version number
```

###### baseline

```sh

$ mysql-migrate baseline 1.0
[2015-12-26 13:38:33.137] [INFO] [MySQL Client /localhost:3306] - Connection pool of 10 connections was established
[2015-12-26 13:38:33.141] [INFO] [SchemaMgr/ myproject] - Creating Schema `myproject` if not exists.
[2015-12-26 13:38:33.160] [INFO] [SchemaMgr/ myproject] - Creating `schema_version` object in `myproject` schema.
[2015-12-26 13:38:33.177] [INFO] [SchemaMgr/ myproject] - Deleting objects in `myproject`.`schema_version`
[2015-12-26 13:38:33.179] [INFO] [SchemaMgr/ myproject] - Saving version `1.0` to `myproject`.`schema_version`
[2015-12-26 13:38:33.183] [INFO] [MySQL Client /localhost:3306] - Shutting down connection pool
[2015-12-26 13:38:33.186] [INFO] console - Exit with status code 0

```


###### info

```sh

$ mysql-migrate info
[2015-12-26 13:39:53.077] [INFO] [MySQL Client /localhost:3306] - Connection pool of 10 connections was established
[2015-12-26 13:39:53.080] [INFO] [SchemaMgr/ myproject] - Reading objects from `myproject`.`schema_version`
[2015-12-26 13:39:53.112] [INFO] console - Schema: `myproject`, Version: 1.0
[2015-12-26 13:39:53.126] [INFO] console - ┌────────┬──────────────┬────────────────┬────────┐
[2015-12-26 13:39:53.126] [INFO] console - │ Script │ Description  │ Execution Time │ Status │
[2015-12-26 13:39:53.126] [INFO] console - ├────────┼──────────────┼────────────────┼────────┤
[2015-12-26 13:39:53.127] [INFO] console - │        │ Base version │ 0 ms           │ OK     │
[2015-12-26 13:39:53.127] [INFO] console - └────────┴──────────────┴────────────────┴────────┘
[2015-12-26 13:39:53.127] [INFO] [MySQL Client /localhost:3306] - Shutting down connection pool
[2015-12-26 13:39:53.128] [INFO] console - Exit with status code 0

```

###### migrate

First, we upload our migration script to the data directory (see config section).
For example, `/etc/mysql-migraterc/data/myproject/v1_1_Create_User_Table.sql` will have
the following statement:

```sql
CREATE TABLE IF NOT EXISTS users (
  name VARCHAR(25) NOT NULL,
  PRIMARY KEY(name)
);
```

Now we are ready to go with the migration script.

```sh

$ mysql-migrate.js migrate
[2015-12-26 14:06:08.730] [INFO] [MySQL Client /localhost:3306] - Connection pool of 10 connections was established
[2015-12-26 14:06:08.734] [INFO] [SchemaMgr/ myproject] - Reading objects from `myproject`.`schema_version`
[2015-12-26 14:06:08.768] [INFO] [MySQL Client /localhost:3306] - Starting transaction
[2015-12-26 14:06:08.795] [INFO] [MySQL Client /localhost:3306] - Committing
[2015-12-26 14:06:08.798] [INFO] [SchemaMgr/ myproject] - Saving version `1.1` to `myproject`.`schema_version`
[2015-12-26 14:06:08.809] [INFO] [MySQL Client /localhost:3306] - Shutting down connection pool
[2015-12-26 14:06:08.811] [INFO] console - Exit with status code 0

```

And we can see the version bump by calling `info` again.

```sh

$ mysql-migrate.js info
[2015-12-26 14:06:57.206] [INFO] [MySQL Client /localhost:3306] - Connection pool of 10 connections was established
[2015-12-26 14:06:57.210] [INFO] [SchemaMgr/ myproject] - Reading objects from `myproject`.`schema_version`
[2015-12-26 14:06:57.238] [INFO] console - Schema: `myproject`, Version: 1.1
[2015-12-26 14:06:57.271] [INFO] console - ┌─────────────────────────────┬───────────────────┬────────────────┬────────┐
[2015-12-26 14:06:57.272] [INFO] console - │ Script                      │ Description       │ Execution Time │ Status │
[2015-12-26 14:06:57.272] [INFO] console - ├─────────────────────────────┼───────────────────┼────────────────┼────────┤
[2015-12-26 14:06:57.272] [INFO] console - │ v1_1__Create_User_Table.sql │ Create User Table │ 32 ms          │ OK     │
[2015-12-26 14:06:57.272] [INFO] console - └─────────────────────────────┴───────────────────┴────────────────┴────────┘
[2015-12-26 14:06:57.272] [INFO] [MySQL Client /localhost:3306] - Shutting down connection pool
[2015-12-26 14:06:57.273] [INFO] console - Exit with status code 0

```

###### repair

Since failures are documented by the schema manager in the same manner
as successful events, it is impossible to fix a failure by running the migration again.
In order to fix a failure, we must call `repair`, which will scan the data directory
for the same script, but this time with the correct syntax, and re-base the version.

Hence, let's change the example above to have a syntax error:

```sql
CREATE TABLE IF NOT EXISTS users (
  name VARCHAR(25) NOT NULL,
  PRIMARY KEY(user_name)
);
```

After trying to run the migration (with no success), there will be no version bump
but we will be able to see the failures of the attempted version.

```sh

$ mysql-migrate info
[2015-12-26 17:10:23.921] [INFO] [SchemaMgr/ myproject] - Reading objects from `myproject`.`schema_version`
[2015-12-26 17:10:24.026] [INFO] console - Schema: `myproject`, Version: 1.0
[2015-12-26 17:10:24.044] [INFO] console - ┌────────────────────────────┬───────────────────┬────────────────┬────────┬─────────────────────────────────────────────────────────────────────────────┐
[2015-12-26 17:10:24.044] [INFO] console - │ Script                     │ Description       │ Execution Time │ Status │ Reason                                                                      │
[2015-12-26 17:10:24.044] [INFO] console - ├────────────────────────────┼───────────────────┼────────────────┼────────┼─────────────────────────────────────────────────────────────────────────────┤
[2015-12-26 17:10:24.044] [INFO] console - │                            │ Base version      │ 0 ms           │ OK     │                                                                             │
[2015-12-26 17:10:24.044] [INFO] console - ├────────────────────────────┼───────────────────┼────────────────┼────────┼─────────────────────────────────────────────────────────────────────────────┤
[2015-12-26 17:10:24.044] [INFO] console - │ v1_1__Create_User_Table.js │ Create User Table │ 15 ms          │ FAILED │ ER_KEY_COLUMN_DOES_NOT_EXITS: Key column 'user_name' doesn't exist in table │
[2015-12-26 17:10:24.045] [INFO] console - └────────────────────────────┴───────────────────┴────────────────┴────────┴─────────────────────────────────────────────────────────────────────────────┘
[2015-12-26 17:10:24.048] [INFO] console - Exit with status code 0

```

Calling `migrate` again at this stage will do nothing since the 
migration tool will ignore any object that was already registered to the schema revision.
`repair` will go over failures and try to run them again.

```sh

$ mysql-migrate repair
[2015-12-26 17:39:58.266] [INFO] [SchemaMgr/ myproject] - Reading objects from `myproject`.`schema_version`
[2015-12-26 17:39:58.350] [INFO] [SchemaMgr/ myproject] - Preparing to repair 1.1/v1_1__Create_User_Table.js
[2015-12-26 17:39:58.360] [INFO] [SchemaMgr/ myproject] - Starting transaction
[2015-12-26 17:39:58.380] [INFO] [SchemaMgr/ myproject] - Committing transaction
[2015-12-26 17:39:58.382] [INFO] [SchemaMgr/ myproject] - Saving version `1.1` to `myproject`.`schema_version`
[2015-12-26 17:39:58.391] [INFO] console - Exit with status code 0

```

If repair completed succesfully, we can now see the version bump.

```sh

$ mysql-migrate info
[2015-12-26 17:40:05.141] [INFO] [SchemaMgr/ myproject] - Reading objects from `myproject`.`schema_version`
[2015-12-26 17:40:05.224] [INFO] console - Schema: `myproject`, Version: 1.1
[2015-12-26 17:40:05.246] [INFO] console - ┌────────────────────────────┬───────────────────┬────────────────┬────────┬────────┐
[2015-12-26 17:40:05.246] [INFO] console - │ Script                     │ Description       │ Execution Time │ Status │ Reason │
[2015-12-26 17:40:05.246] [INFO] console - ├────────────────────────────┼───────────────────┼────────────────┼────────┼────────┤
[2015-12-26 17:40:05.246] [INFO] console - │ v1_1__Create_User_Table.js │ Create User Table │ 30 ms          │ OK     │        │
[2015-12-26 17:40:05.247] [INFO] console - └────────────────────────────┴───────────────────┴────────────────┴────────┴────────┘
[2015-12-26 17:40:05.247] [INFO] console - Exit with status code 0

```

###### clean

```sh

$ mysql-migrate clean
[2015-12-26 13:26:18.042] [INFO] [MySQL Client /localhost:3306] - Connection pool of 10 connections was established
[2015-12-26 13:26:18.045] [INFO] [SchemaMgr/ myproject] - Dropping objects in `myproject`
[2015-12-26 13:26:18.070] [INFO] [MySQL Client /localhost:3306] - Shutting down connection pool
[2015-12-26 13:26:18.072] [INFO] console - Exit with status code 0

```


#### Configuration
 
Migration tool uses rc file for its settings.

Configuration file should be placed in one of the following locations:

    * $HOME/.mysql-migraterc
    * $HOME/.mysql-migrate/config
    * $HOME/.config/mysql-migrate
    * $HOME/.config/mysql-migrate/config
    * /etc/mysql-migraterc
    * /etc/mysql-migrate/config

Configuration should be in one of the following formats:

    * INI
    * JSON

## INI (example)

```
[logging]
level       =   INFO|DEBUG|WARN|ERROR
filepath    =   /path/to/logfile

[db]
host        =   localhost
user        =   root
password    =   pass

[migration]
schema      =   myproject
datadir     =   /etc/mysql-migraterc/data
```


## JSON (example)

```javascript
{
    "logging": {
        "level": "INFO",
        "filepath": null
    },
    "db": {
        "host": "localhost",
        "user": "root",
        "password": ""
    },
    "migration": {
        "schema": "myproject",
        "datadir": "/etc/mysql-migraterc/data/myproject"
    }
}
```