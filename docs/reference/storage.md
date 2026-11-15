# Storage

This page explains where Door+ saves its data, how to set up SQLite or MySQL/MariaDB, how to share one database between servers, and how backups and updates work. It is for server administrators.

Door+ saves doors, access entries, triggers, groups and known player names in a database. Everything is kept in memory while the server runs; changes are written to the database in the background.

## Choosing a storage type

| | SQLite (default) | MySQL / MariaDB |
|---|---|---|
| Setup | Nothing to do | Needs a database server |
| Where the data is | `plugins/DoorPlus/doorplus.db` | On your database server |
| Several servers on one database | No | Yes (no live sync, see below) |
| Automatic backup before updates | Yes | No: make your own |

SQLite is the right choice for a single server. Use MySQL or MariaDB if you already run one, or if several servers should keep their doors in the same place.

!!! warning "Storage changes need a restart"
    Every key under `storage` is read only when the server starts. `/dp admin reload` does not apply them and prints `Changes to the 'storage' section require a server restart`.

## storage keys

| Key | Default | Allowed values | Meaning |
|---|---|---|---|
| `storage.type` | `sqlite` | `sqlite`, `mysql`, `mariadb` | Database type. |
| `storage.server-id` | `default` | 1 to 32 characters | Name of this server inside a shared database. |
| `storage.table-prefix` | `dp_` | 1 to 16 letters, digits or `_` | Prefix of every table name. |
| `storage.sqlite.file` | `doorplus.db` | file name | SQLite file inside `plugins/DoorPlus/`. |
| `storage.mysql.host` | `localhost` | | Database server address. |
| `storage.mysql.port` | `3306` | `1` to `65535` | Database server port. |
| `storage.mysql.database` | `doorplus` | | Database name. It must already exist. |
| `storage.mysql.username` | `doorplus` | | Database user. |
| `storage.mysql.password` | `''` | | Password of the user. |
| `storage.mysql.properties` | `'useSSL=false'` | | Extra connection options, `key=value` pairs joined with `&`. |
| `storage.pool.max-size` | `4` | `1` to `64` | Maximum open connections (MySQL/MariaDB). SQLite always uses one. |
| `storage.pool.connection-timeout-ms` | `10000` | `250` to `600000` | How long to wait for a connection, in milliseconds. |

An invalid value is replaced by its default with a console warning (see [Configuration](configuration.md#invalid-values)). For example an invalid `storage.type` falls back to `sqlite`.

## SQLite setup

SQLite works without any setup.

1. Leave `storage.type: sqlite`.
2. Start the server. Door+ creates `plugins/DoorPlus/doorplus.db` and its tables.

Because SQLite runs in WAL mode, you may also see `doorplus.db-wal` and `doorplus.db-shm` next to the file while the server runs. They belong to the database: never delete them while the server is running.

## MySQL / MariaDB setup

1. On your database server, create a database and a user. Example for MySQL or MariaDB:

    ```sql
    CREATE DATABASE doorplus CHARACTER SET utf8mb4;
    CREATE USER 'doorplus'@'%' IDENTIFIED BY 'choose-a-password';
    GRANT ALL PRIVILEGES ON doorplus.* TO 'doorplus'@'%';
    FLUSH PRIVILEGES;
    ```

    Door+ creates and updates its own tables, so the user needs the rights to create and alter tables.

2. Stop the Minecraft server.
3. Edit `plugins/DoorPlus/config.yml`:

    ```yaml
    storage:
      type: mysql            # or mariadb
      server-id: default
      table-prefix: dp_
      mysql:
        host: 127.0.0.1
        port: 3306
        database: doorplus
        username: doorplus
        password: 'choose-a-password'
        properties: 'useSSL=false'
      pool:
        max-size: 4
        connection-timeout-ms: 10000
    ```

4. Start the server.
5. Check the console. If the connection works, Door+ starts normally and `/dp admin version` shows the storage type. If it fails, the console shows `Cannot open the database (mysql): <reason>; disabling Door+`, and every `/dp` command answers **Door+ is disabled. Details are in the server console.** Fix the settings and restart.

- `type: mysql` connects with the MySQL driver, `type: mariadb` with the MariaDB driver. Use the one that matches your server.
- Tables are created with the InnoDB engine and the `utf8mb4` character set.
- If your database requires SSL, change `properties`, for example `'useSSL=true&requireSSL=true'`. The accepted options depend on the driver.

!!! note "Drivers"
    Paper already contains the SQLite and MySQL drivers. The MariaDB driver and the connection pool are listed as libraries in Door+, and Paper downloads them on the first start, so that start needs internet access.

## Sharing one database between servers

Several servers (for example in a BungeeCord or Velocity network) can use the same MySQL/MariaDB database:

1. Point every server to the same database with the same `table-prefix`.
2. Give each server a **different** `storage.server-id`, for example `survival`, `creative`, `lobby`.
3. Restart the servers.

How it works:

- Every door and every group is saved with the `server-id` of the server that created it. Each server loads **only its own** doors and groups (with their access entries and triggers).
- Door names and group names are unique per owner **and per server**: the same player can have a door called `gate` on two servers.
- Known player names are shared by all servers.
- There is **no live synchronisation**. Servers never see or move each other's doors. Sharing a database only means one place to store and back up the data.

!!! warning "Do not use the same server-id on two running servers"
    Two servers with the same `server-id` load the same doors and overwrite each other's changes. Use the same `server-id` only for the same server.

!!! warning "Changing server-id later"
    The doors stay saved under the old `server-id`. After a change, the server loads no doors until you set the old value back.

## Table prefix

All tables start with `storage.table-prefix` (default `dp_`, for example `dp_doors`, `dp_triggers`, `dp_meta`).

- Use a prefix if the database also holds other plugins' tables.
- Two independent sets of Door+ data can live in one database with different prefixes.
- Changing the prefix later makes Door+ create a new, empty set of tables. The old tables are not renamed or copied.

## Backups

=== "SQLite"

    1. Stop the server.
    2. Copy `plugins/DoorPlus/doorplus.db` (and `doorplus.db-wal` / `doorplus.db-shm` if they exist) to a safe place.
    3. Start the server.

    To restore, stop the server, put the copied files back, and start it.

=== "MySQL / MariaDB"

    Use your database tools, for example:

    ```
    mysqldump -u doorplus -p doorplus > doorplus-backup.sql
    ```

    Back up the whole database, or every table that starts with your `table-prefix`.

Door+ also makes an automatic backup of SQLite before it changes the database format (see below). It does not do this for MySQL/MariaDB.

## Updates and database migrations

The database has a format version (stored in the `meta` table). When a new Door+ version needs a newer format, it upgrades the database automatically on start:

1. **SQLite only:** before changing anything, Door+ copies the database to `doorplus.db.bak-v<n>`, where `<n>` is the old format version (for example `doorplus.db.bak-v1`).
2. Each upgrade step runs in a transaction, so a failed step leaves the database as it was.
3. The console shows `Database schema migrated to version <n>` for each step.

If the database was written by a **newer** Door+ version than the one installed (for example after a downgrade), Door+ does not open it. The console shows `Database schema version <x> is newer than the supported version <y>; update Door+ or restore a backup` and Door+ disables itself. Install the newer version again, or restore a backup made before the update (for SQLite, the `.bak-v<n>` file: stop the server, rename it to `doorplus.db`, start the server).

!!! tip "MySQL/MariaDB users"
    Make a backup before every Door+ update. Door+ cannot back up a MySQL/MariaDB database for you.

## Write errors and shutdown

- Writes are done in the background in the order they happen. A failed write is retried 3 times (after 0.1, 0.5 and 2 seconds); if it still fails, the error is written to the console as `Database write failed after retries: ...`.
- On server stop, Door+ waits up to 10 seconds for pending writes. If they are not done in time, the console shows `Pending database writes not flushed within 10 seconds`.

## Switching storage type

Door+ has **no built-in tool** to move data from SQLite to MySQL/MariaDB or back.

What happens if you change `storage.type` and restart:

- Door+ connects to the new database and creates empty tables there. Your doors are **not** copied: the server starts with no doors.
- The old data is not deleted. If you switch back, the old doors are loaded again.
- Doors built in the meantime stay in the database they were created in.

Decide on the storage type before players start building. If you must switch later, copying the tables between databases is a manual database task and is not covered by Door+.
