# Installation

This page explains how to install, update and remove Door+ on a server. It is for server owners and
administrators.

## Requirements

| Requirement | Supported |
|---|---|
| Server software | **Paper** 1.21.4 up to 26.x |
| Java | Java 21 or newer (Paper 26.x itself needs Java 25) |
| Players | Java Edition clients |

Door+ does **not** support:

- Spigot or CraftBukkit (Door+ uses Paper-only features).
- Folia.
- Bedrock players joining through Geyser.

Optional plugins that Door+ can use when they are installed: LuckPerms, PlaceholderAPI, WorldGuard,
ItemsAdder, Nexo, Oraxen, CraftEngine, Citizens and FancyNpcs. See [Integrations](../integrations/index.md).
None of them is required.

!!! note "Download from the marketplace"
    Use the jar you downloaded from the marketplace where you bought Door+. A release jar checks the
    purchase with that marketplace when the server starts. See
    [Licence, verification and privacy](../help/licence-and-privacy.md).

## Install

1. Stop the server.
2. Put the Door+ jar in the `plugins` folder of the server.
3. Start the server.
4. Check the console: Door+ prints no error when it starts correctly.
5. In game, as an operator, type `/dp admin version`. You see the plugin version, the server version,
   the storage type and the active integrations.

!!! warning "Do not use `/reload`"
    Install, update and remove Door+ with a full server restart.

## What happens on the first start

On the first start Paper downloads two libraries that Door+ lists in its `plugin.yml`: the HikariCP
connection pool and the MariaDB database driver. This happens once; the server needs internet access for it.

Door+ then creates the folder `plugins/DoorPlus` with these files:

| File | Content |
|---|---|
| `config.yml` | The configuration. See [Configuration](../reference/configuration.md). |
| `lang/en.yml`, `lang/it.yml` | The English and Italian messages. See [Language files](../reference/language.md). |
| `doorplus.db` | The SQLite database with doors, access entries, triggers and groups (default storage). See [Storage](../reference/storage.md). |

A release jar also writes a small hidden file `.license` with the state of the purchase check.

## Let players create doors

With the default permissions every player can use, open, close and manage the doors on which they have a
role, but **only operators can create doors**. The creation permissions default to op.

Give your players one of these permissions:

| Permission | Effect |
|---|---|
| `doorplus.create.*` | Create doors of every type. |
| `doorplus.create.<type>` | Create doors of one type, for example `doorplus.create.big_door`. |

The type ids are `big_door`, `drawbridge`, `portcullis`, `sliding_door`, `garage_door`, `split_door`,
`folding_door`, `rolling_shutter`, `platform`, `revolving_door`, `windmill`, `flag` and `clock`.

For example, with LuckPerms, to let every player create big doors:

```
/lp group default permission set doorplus.create.big_door true
```

Other things you may want to change:

- `doorplus.command.teleport` (default op) lets players teleport to their doors.
- Each player can own at most 10 doors of at most 500 blocks by default. Change `limits.doors-per-player`
  and `limits.max-door-blocks` in `config.yml`, or give per-player limits with permissions. See
  [Limits](../reference/limits.md).

The full list is on the [Permissions](../reference/permissions.md) page.

## Update to a new version

1. Download the new jar from the marketplace.
2. Stop the server.
3. Replace the old Door+ jar in `plugins` with the new one. Keep the `plugins/DoorPlus` folder.
4. Start the server.

On start, Door+ updates your files by itself:

- **Configuration**: keys added in the new version are written into your `config.yml` with their default
  value and comment. Your values are kept. An invalid value does not stop the plugin: it is logged in the
  console and its default is used.
- **Language files**: your files in `lang/` are never overwritten. A message missing from your file is taken
  from the built-in file of the new version.
- **Database**: if the new version needs a newer database layout, Door+ upgrades it. With SQLite it first
  copies the database to `doorplus.db.bak-v<old version>` in the same folder.

!!! warning "Going back to an older version"
    An older Door+ refuses to start on a database already upgraded by a newer version ("Database schema
    version ... is newer than the supported version"). To downgrade with SQLite, restore the
    `doorplus.db.bak-v<version>` copy made before the upgrade.

## Uninstall

1. If you want to keep the data for later, copy the `plugins/DoorPlus` folder somewhere safe.
2. Stop the server. Door+ finishes every running animation and saves its data while the server stops.
3. Delete the Door+ jar from `plugins`.
4. Delete the `plugins/DoorPlus` folder if you do not need the data.
5. Start the server.

The door blocks are normal blocks in the world. After the uninstall they stay exactly where they were when
the server stopped: an open door stays open. If you want every door closed, close them before you stop the
server.
