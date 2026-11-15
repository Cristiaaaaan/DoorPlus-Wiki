# Commands

This page lists every Door+ command with its exact syntax, the permission it needs, the role you need on the door, and examples. It is for players and staff who prefer commands to menus, and for administrators who run commands from the console.

## Basics

- The root command is `/doorplus`. The aliases `/dp` and `/door` do exactly the same thing. This page uses `/dp`.
- Syntax: `<required>`, `[optional]`, `a|b` means "type one of these".
- Subcommands, options and values are not case-sensitive (`/dp OPEN gate` works).
- In enum values you can type `-` or `_` (`owner-only` and `owner_only` are the same).
- Yes/no values accept `true`, `on`, `yes` or `false`, `off`, `no`.
- `/dp` with no arguments opens the main menu if you have `doorplus.command.menu`. Otherwise (and in the console) it shows the help.
- Tab completion only suggests the subcommands, doors, groups and values you are allowed to use.
- Every command checks the **permission** first, then your **role** on the door. Roles are explained in [Privacy and roles](../access/privacy-and-roles.md). For all permission nodes see [Permissions](permissions.md).

!!! note "Unknown or wrong commands"
    An unknown subcommand answers **Unknown subcommand. Type /door help for the list.** A command with wrong arguments answers **Usage:** followed by the correct syntax.

## Door references

Wherever a command asks for `<door>`, you can type the door in three ways:

| Form | Example | Meaning |
|---|---|---|
| `#<id>` | `#12` | The door with that numeric ID. Always works. You see the ID in `/dp list` and `/dp info`. |
| `<name>` | `castle_gate` | Your own door with that name. |
| `<owner>:<name>` | `Steve:castle_gate` | A door owned by another player. |

Rules:

- Door names are 1 to 32 characters: letters, digits, `_` and `-`. Names are not case-sensitive and are unique per owner, not per server.
- You can only refer to doors on which you have a role (User, Admin or Owner), unless you have a bypass permission (`doorplus.admin.bypass.use`, `.edit` or `.manage`). Any other door answers **Door not found.**, so nobody can discover doors they have no access to.
- The console, and players with a bypass permission, can also use a bare `<name>` for another player's door if exactly one door on the server has that name. If several doors share the name, Door+ lists them, for example **More doors are called gate: #5 (Mario), #9 (Luca). Use #id.** Repeat the command with `#id`.
- Tab completion shows your own doors first, then `owner:name` for doors shared with you. In the console it shows `#id`.

**Group references** (`<group>`) work the same way: `#<id>`, `<name>` (your own group) or `<owner>:<name>`. You can only use your own groups unless you have `doorplus.admin.bypass.manage`.

!!! warning "Groups in the console"
    The console has no "own" groups, so a bare group name does not work there. Use `#<id>` or `<owner>:<name>` (for example `/dp group open #3`). `/dp group list` in the console shows every group with its ID.

**Trigger IDs** (`<trigger id>`) are numbers, with or without `#`: `12` or `#12`. You see them in `/dp trigger list`.

## Quick reference

| Command | Permission | Role on the door |
|---|---|---|
| `/dp help [page]` | none | none |
| `/dp menu [<door>]` | `doorplus.command.menu` | any role to open a door menu |
| `/dp create <type> [name]` | `doorplus.create.<type>` | none |
| `/dp cancel` | none | none |
| `/dp list [player]` | `doorplus.command.list` (+ `doorplus.admin.list` for another player) | none |
| `/dp info <door>` | `doorplus.command.info` | any role |
| `/dp toggle\|open\|close <door> [door...]` | `doorplus.command.toggle` | decided by the door's access rules |
| `/dp goto <door> <stop>` | `doorplus.command.toggle` | decided by the door's access rules |
| `/dp lock\|unlock <door>` | `doorplus.command.manage` | Admin |
| `/dp rename <door> <name>` | `doorplus.command.manage` | Admin |
| `/dp redefine <door>` | `doorplus.command.manage` | Admin |
| `/dp set <door> <option> <value>` | `doorplus.command.manage` | Admin |
| `/dp privacy <door> <private\|restricted\|public>` | `doorplus.command.manage` | Owner |
| `/dp transfer <door> <player>` | `doorplus.command.manage` | Owner |
| `/dp delete <door> [confirm]` | `doorplus.command.manage` | Owner |
| `/dp tp <door>` | `doorplus.command.teleport` | any role |
| `/dp access ...` | `doorplus.command.manage` | Owner |
| `/dp trigger ...` | `doorplus.command.manage` (+ `doorplus.trigger.<kind>` for `add`) | Admin (or owner of the group) |
| `/dp key ...` | `doorplus.command.manage` | Admin |
| `/dp group toggle\|open\|close <group>` | `doorplus.command.toggle` | owner of the group |
| `/dp group ...` (other actions) | `doorplus.command.manage` | owner of the group; Admin on the door for `add` |
| `/dp admin ...` | one `doorplus.admin.<action>` node per action | none |

"Any role" means User, Admin or Owner, or a bypass permission. Players with `doorplus.admin.bypass.edit` act as Admin on every door, and players with `doorplus.admin.bypass.manage` act as Owner on every door. The console always acts as Owner.

## General commands

### /dp help

```
/dp help [page]
```

Shows the commands **you** are allowed to use, 8 per page. `page` is a number from 1 to the last page. The last line tells you how to open the next page.

- Permission: none.
- Example: `/dp help 2`

### /dp menu

```
/dp menu [<door>]
```

Without a door, opens the main menu. With a door, opens the menu of that door. See [Menus](../doors/menus.md).

- Permission: `doorplus.command.menu`.
- Players only.
- Example: `/dp menu castle_gate`

### /dp create

```
/dp create <type> [name]
```

Starts the guided creation of a new door. You receive the creation stick and follow the instructions in chat. See [Creating a door](../doors/creating.md).

- `type`: one of `big_door`, `drawbridge`, `portcullis`, `sliding_door`, `garage_door`, `split_door`, `folding_door`, `rolling_shutter`, `platform`, `revolving_door`, `windmill`, `flag`, `clock`. Tab completion only shows the types you may create and that are enabled in `types.<type>.enabled`.
- `name` (optional): 1 to 32 letters, digits, `_` or `-`. If you leave it out, the creation asks for it.
- Permission: `doorplus.create.<type>`, for example `doorplus.create.big_door`.
- Players only.
- Fails if the type is disabled in the configuration, if you reached your door limit (see [Limits](limits.md)), if you already have a creation in progress, or if your inventory has no free slot for the creation stick.
- Examples: `/dp create big_door`, `/dp create sliding_door shop_door`

### /dp cancel

```
/dp cancel
```

Cancels what you are doing right now, in this order: a door creation, then a trigger selection or inspection, then a chat input (for example a name you were asked to type). If nothing is in progress you see **There is nothing to cancel.**

- Permission: none.
- Players only.

## Using doors

### /dp list

```
/dp list [player]
```

- Without a player: lists your own doors, then **Doors shared with** you (doors where you have a role).
- With a player name: lists the doors owned by that player. Needs `doorplus.admin.list` as well (typing your own name does not).
- In the console without a player: lists **all** doors of the server.
- Each line shows `#id`, the door reference, the type, the state and the privacy.
- Permission: `doorplus.command.list`.
- Examples: `/dp list`, `/dp list Steve`

### /dp info

```
/dp info <door>
```

Shows the details of a door: type, state, owner, your role, location, size, number of blocks, direction, privacy, locked, redstone, protected blocks, region access, auto-close, speed, collisions, key mode, number of access entries, number of triggers and how many times it was operated.

- Permission: `doorplus.command.info`.
- Example: `/dp info #12`

### /dp toggle, /dp open, /dp close

```
/dp toggle <door> [door...]
/dp open <door> [door...]
/dp close <door> [door...]
```

Operates one or more doors. `toggle` opens a closed door and closes an open door; `open` and `close` only move the door if it is not already in that state. You can list several doors in one command.

- Permission: `doorplus.command.toggle`.
- Role: a player goes through the normal access check of the door (privacy, locked, key, cooldown), exactly as if they used it in the world. See [Privacy and roles](../access/privacy-and-roles.md).
- Console: the access check is skipped. The console sees **Door ... is moving.** for each door that starts moving.
- Examples: `/dp open castle_gate`, `/dp close gate_left gate_right`, `/dp toggle #12`

### /dp goto

```
/dp goto <door> <stop>
```

Sends a platform to a stop. The first stop is `0`, the last is the number of stops minus 1. Only works on doors of type `platform`; other doors answer **... is not a platform.**

- Permission: `doorplus.command.toggle`.
- Role: same access check as `/dp toggle`. The console skips it.
- Example: `/dp goto elevator 2`

### /dp tp

```
/dp tp <door>
```

Teleports you next to the door, facing it. If the door's world is not loaded you see **The world ... of ... is not loaded.**

- Permission: `doorplus.command.teleport` (default: operators only).
- Players only.
- Example: `/dp tp castle_gate`

## Managing doors

### /dp lock, /dp unlock

```
/dp lock <door>
/dp unlock <door>
```

Locks or unlocks a door. A locked door cannot be operated (see [Privacy and roles](../access/privacy-and-roles.md)).

- Permission: `doorplus.command.manage`. Role: Admin.
- Example: `/dp lock vault_door`

### /dp rename

```
/dp rename <door> <name>
```

Renames a door. The new name follows the name rules (1-32 letters, digits, `_`, `-`) and must not be used by another door of the same owner.

- Permission: `doorplus.command.manage`. Role: Admin.
- Example: `/dp rename gate main_gate`

### /dp redefine

```
/dp redefine <door>
```

Starts a guided session to select the region of the door again. See [Editing a door](../doors/editing.md).

- Permission: `doorplus.command.manage`. Role: Admin.
- Players only.
- Example: `/dp redefine castle_gate`

### /dp set

```
/dp set <door> <option> <value>
/dp set <door> base-state
```

Changes one setting of a door. On success you see **<option> of <door> set to <value>.**

- Permission: `doorplus.command.manage`. Role: Admin.

| Option | Values | Notes |
|---|---|---|
| `direction` | a direction allowed for this door: `cw`, `ccw`, `up`, `down`, `north`, `east`, `south`, `west`, `none` | The door must be closed and still. Tab completion shows only the directions the door's type accepts. |
| `blocks-to-move` | `0` to `limits.max-blocks-to-move` | `0` = the type's automatic value. Only for types that move by a number of blocks. The door must be closed and still. |
| `auto-close` | seconds, `0` to `limits.max-auto-close-seconds` | `0` = no automatic closing. |
| `speed` | a decimal between `limits.speed-multiplier.min` and `.max` | Speed multiplier, `1.0` = normal speed. |
| `collision` | `default`, or a mode listed in `collision.allowed-modes`: `none`, `shulker`, `push` | `default` uses `collision.default-mode`. |
| `redstone` | `true` / `false` | Whether redstone can operate the door. |
| `protect` | `true` / `false` | Protects the door's blocks from being broken. |
| `region-access` | `true` / `false` | Members of the WorldGuard region can use the door. If WorldGuard is not active you get a warning. See [WorldGuard](../integrations/worldguard.md). |
| `base-state` | no value | Makes the door's current position its closed state. |

Some door types add their own options:

| Option | Door type |
|---|---|
| `mode` | `split_door` |
| `visible-rows` | `rolling_shutter` |
| `stops` | `platform` |
| `quarter-turns` | `revolving_door` |
| `degrees-per-second` | `windmill` |
| `time-source` | `clock` |

Values and meaning of these options are explained on each type's page under [Door types](../doors/types/index.md). Tab completion suggests valid values. An option the door does not have answers **Unknown option ... for this door.**

Examples:

```
/dp set castle_gate auto-close 10
/dp set castle_gate speed 1.5
/dp set castle_gate collision push
/dp set shop_door redstone true
/dp set lift stops 0,5,12
/dp set castle_gate base-state
```

### /dp privacy

```
/dp privacy <door> <private|restricted|public>
```

Sets who can use the door: `private` (owner and admins), `restricted` (owner, admins and every access entry), `public` (everyone).

- Permission: `doorplus.command.manage`. Role: Owner.
- Example: `/dp privacy castle_gate restricted`

### /dp transfer

```
/dp transfer <door> <player>
```

Gives the door to another player, who becomes its Owner. The player must be known to the server (has joined before).

- Permission: `doorplus.command.manage`. Role: Owner.
- Fails if the new owner already has a door with the same name.
- When a player runs it (without `doorplus.admin.bypass.limits`), the new owner must have room in their door limit. For an offline recipient the `limits.doors-per-player` value is used. The console does not check the limit.
- Example: `/dp transfer castle_gate Alex`

### /dp delete

```
/dp delete <door> [confirm]
```

Deletes a door. Without `confirm`, Door+ only asks you to confirm: **Delete ... for good? Type /door delete #12 confirm to confirm.** Run it again with `confirm` to delete.

- Permission: `doorplus.command.manage`. Role: Owner.
- Example: `/dp delete #12 confirm`

## Access entries

Access entries say who may use a door when its privacy is `restricted`, and who is Admin. See [Privacy and roles](../access/privacy-and-roles.md).

All access commands need `doorplus.command.manage` and the **Owner** role on the door.

### /dp access list

```
/dp access list <door>
```

Lists the entries of a door with type, subject and role, and shows the door's privacy.

- Example: `/dp access list castle_gate`

### /dp access add

```
/dp access add <door> <player|group|permission> <subject> [user|admin]
```

Adds an entry, or changes the role of an existing entry. The role is `user` if you leave it out. `owner` is not accepted.

- `player`: a player name known to the server.
- `group`: a LuckPerms group name. Needs `doorplus.access.group` and LuckPerms installed.
- `permission`: a permission node; any player with that node gets the entry. Needs `doorplus.access.permission`.
- The list is limited by `limits.max-access-entries` (see [Limits](limits.md)).
- Examples: `/dp access add castle_gate player Alex`, `/dp access add castle_gate group guards admin`, `/dp access add castle_gate permission myserver.castle`

### /dp access remove

```
/dp access remove <door> <player|group|permission> <subject>
```

Removes an entry. Tab completion lists the current entries of the door.

- Example: `/dp access remove castle_gate player Alex`

## Triggers

Triggers are blocks, redstone, entities or areas that operate a door or a group. See [Triggers](../access/triggers.md).

All trigger commands need `doorplus.command.manage`. A target is a `<door>` or `group:<group>`. You need the **Admin** role on the door, or you must own the group (or have `doorplus.admin.bypass.manage`). A trigger you cannot edit answers **Trigger not found.**

### /dp trigger list

```
/dp trigger list <door|group:group>
```

Lists the triggers of a door or a group with ID, type, position, action, access mode and whether it is enabled.

- Examples: `/dp trigger list castle_gate`, `/dp trigger list group:castle`

### /dp trigger add

```
/dp trigger add <door|group:group> <block|redstone|entity|proximity> [action] [radius]
```

Starts the selection mode: click the block or entity you want to link. For `proximity` you choose the centre instead. Type `/dp cancel` to stop. The selection times out after `triggers.selection-timeout-seconds`.

- `action` (optional): `toggle`, `open`, `close`, `follow` or `goto`.
- `radius` (optional, `proximity` only): `1` to `limits.max-proximity-radius`.
- Extra permission: `doorplus.trigger.<kind>`, for example `doorplus.trigger.block`.
- Players only.
- Custom blocks, furniture and NPC triggers are linked from the menu, not with this command.
- Examples: `/dp trigger add castle_gate block`, `/dp trigger add castle_gate block open`, `/dp trigger add group:castle proximity toggle 5`

### /dp trigger remove

```
/dp trigger remove <trigger id>
```

Removes a trigger.

- Example: `/dp trigger remove 12`

### /dp trigger set

```
/dp trigger set <trigger id> <option> <value>
```

| Option | Values | Applies to |
|---|---|---|
| `action` | `toggle`, `open`, `close`, `follow`, `goto` (only the actions valid for this trigger and its target) | all |
| `arg` | stop number, from `0` | only when the action is `goto` |
| `access` | `inherit`, `public`, `restricted`, `owner-only` | all except redstone |
| `cooldown` | ticks, `0` to `72000` (20 ticks = 1 second) | all |
| `cancel-vanilla` | `true` / `false` | all except redstone and proximity |
| `enabled` | `true` / `false` | all |
| `radius` | `1` to `limits.max-proximity-radius` | proximity only |
| `exit-delay` | ticks, `0` to `72000` | proximity only |

An option that does not fit the trigger answers **The option ... does not apply to this trigger.**

- Examples: `/dp trigger set 12 action open`, `/dp trigger set 12 cooldown 40`, `/dp trigger set 7 radius 6`

### /dp trigger access

```
/dp trigger access <trigger id> <add|remove> <player|group|permission> <subject>
```

Edits the trigger's own access entries (used when the trigger's access mode is `restricted`). Adding a `group` needs `doorplus.access.group` and LuckPerms; adding a `permission` needs `doorplus.access.permission`.

- Example: `/dp trigger access 12 add player Alex`

### /dp trigger inspect

```
/dp trigger inspect
```

Starts the inspection mode: click a block or entity to see the triggers linked to it. Type `/dp cancel` to stop.

- Players only.

## Keys

Keys are items that open a door. See [Keys](../access/keys.md).

All key commands need `doorplus.command.manage` and the **Admin** role on the door.

| Command | What it does |
|---|---|
| `/dp key mode <door> <none\|alternative\|required>` | Sets how the key is used. |
| `/dp key create <door>` | Turns the item in your main hand into a key of the door. Players only. |
| `/dp key custom <door>` | Uses the custom item (ItemsAdder, Nexo, Oraxen, CraftEngine) in your hand as the key. Players only. |
| `/dp key give <door> [player]` | Gives a copy of the key to an online player, or to yourself if you leave the player out. The console must name a player. |
| `/dp key consume <door> <true\|false>` | Whether one key is used up each time the door is opened with it. |
| `/dp key reset <door>` | Generates a new key: all old keys stop working. |
| `/dp key remove <door>` | Removes the key of the door. |

Examples:

```
/dp key create vault_door
/dp key mode vault_door required
/dp key give vault_door Alex
```

## Groups

A group operates several doors together. See [Groups](../access/groups.md).

Operating a group (`toggle`, `open`, `close`) needs `doorplus.command.toggle`. Every other group action needs `doorplus.command.manage`. You can only use your own groups unless you have `doorplus.admin.bypass.manage`.

| Command | What it does |
|---|---|
| `/dp group create <name>` | Creates a group. Name rules as for doors. Limited by `limits.max-groups-per-player`. Players only. |
| `/dp group delete <group>` | Deletes a group. The doors are not deleted. |
| `/dp group rename <group> <name>` | Renames a group. |
| `/dp group add <group> <door>` | Adds a door. You need the Admin role on the door. Limited by `limits.max-doors-per-group`. |
| `/dp group remove <group> <door>` | Removes a door, even one on which you lost your role. |
| `/dp group list` | Lists your groups with ID and number of doors. In the console: every group. |
| `/dp group info <group>` | Shows the owner and the doors of a group. |
| `/dp group toggle\|open\|close <group>` | Operates every door of the group. Players go through each door's access check; the console skips it and gets a summary **Group ...: x/y doors moving.** |

Examples:

```
/dp group create castle
/dp group add castle gate_left
/dp group add castle gate_right
/dp group open castle
```

## Administration

Each admin action has its own permission. All of them are children of `doorplus.admin.*` and default to operators.

| Command | Permission | What it does |
|---|---|---|
| `/dp admin reload` | `doorplus.admin.reload` | Reloads `config.yml` and the language files. Changes to `storage` need a restart. See [Configuration](configuration.md#reload). |
| `/dp admin pause` | `doorplus.admin.pause` | Pauses every door movement; run it again to resume. While paused, nobody can operate doors. The pause is not saved: a restart resumes movements. |
| `/dp admin stopall` | `doorplus.admin.stopall` | Finishes every running animation instantly and reports how many. |
| `/dp admin cleanup` | `doorplus.admin.cleanup` | Removes orphan animation entities left in the world and reports how many. |
| `/dp admin import bigdoors [file] [--dry-run]` | `doorplus.admin.import` | Imports the doors of BigDoors. Without `file` it reads `plugins/BigDoors/doorDB.db`. A relative path is looked up in the `plugins` folder. `--dry-run` only checks the file and reports, without importing. See [Importing from BigDoors](../getting-started/bigdoors-import.md). |
| `/dp admin version` | `doorplus.admin.version` | Shows plugin and API version, server version, storage type, animation engine, active integrations, hooks, number of doors, triggers and groups, and the licensee. |
| `/dp admin debug <door>` | `doorplus.admin.debug` | Writes the internal state of a door to the server console (for bug reports). |
| `/dp admin claim <door>` | `doorplus.admin.claim` | You become the Owner of the door; the previous owner loses it. Players only. A bare name is searched among **every** door of the server; if several doors have it, use `#id`. |

Examples:

```
/dp admin import bigdoors --dry-run
/dp admin import bigdoors BigDoors/doorDB.db
/dp admin claim #12
```

## Console usage

The console always acts as Owner of every door and sees every door. Door references in the console: `#id`, `owner:name`, or a bare name that only one door on the server uses.

| Works in the console | Players only |
|---|---|
| `help`, `list` (no argument = all doors), `info` | `menu` (also `/dp` alone opens the help in the console) |
| `toggle`, `open`, `close`, `goto` (access check skipped) | `create`, `cancel`, `redefine`, `tp` |
| `lock`, `unlock`, `rename`, `set`, `privacy`, `transfer` (no limit check), `delete` | `trigger add`, `trigger inspect` |
| `access list`, `access add`, `access remove` | `key create`, `key custom`, `key give` without a player |
| `trigger list`, `trigger remove`, `trigger set`, `trigger access` | `group create` |
| `key mode`, `key give <door> <player>`, `key consume`, `key reset`, `key remove` | `admin claim` |
| `group delete`, `rename`, `add`, `remove`, `list`, `info`, `toggle`, `open`, `close` (use `#id` or `owner:name`) | |
| `admin reload`, `pause`, `stopall`, `cleanup`, `import`, `version`, `debug` | |

A player-only command run from the console answers **Only players can do this.**

!!! tip "Command blocks and other plugins"
    Command blocks and plugins that run commands as the console can use the console forms, for example `dp open #12`. This is a simple way to open doors from other plugins' events.
