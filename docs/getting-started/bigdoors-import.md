# Coming from BigDoors

This page explains how to move your existing BigDoors doors into Door+ with one command. It is for server
administrators who are replacing BigDoors (version 1) with Door+.

The import reads the BigDoors database, creates the same doors in Door+ with their owners, co-owners and power
blocks, and writes a report. It never changes the BigDoors database: Door+ opens it read-only.

## Before you start

| Requirement | Why |
|---|---|
| BigDoors **version 1** database (`doorDB.db`, SQLite) | This is the only format Door+ can read. |
| The database uses the **current BigDoors schema** (schema version `9`) | Older databases are refused. Start the latest BigDoors v1 release once: it updates its own database. |
| Permission `doorplus.admin.import` | Needed to run the command (operators have it by default). |
| The worlds of your doors are loaded | Doors in unloaded worlds are still imported, but Door+ cannot read the world to pick a default opening direction for them (see [What is mapped](#what-is-mapped)). |

!!! warning "Never run BigDoors and Door+ on the same doors"
    Two plugins moving the same blocks corrupt them. Remove BigDoors as soon as the import is done, before
    players use the doors. If BigDoors is still enabled when you start the import, Door+ reminds you:
    **BigDoors is enabled. Remove it after the migration: two plugins moving the same blocks corrupt them.**

## Step by step

1. Update BigDoors to its latest v1 release, start the server once, then stop it. This brings the database to
   schema version `9`.
2. Make a copy of `plugins/BigDoors/doorDB.db` somewhere safe (good practice before any migration).
3. Install Door+ (see [Installation](installation.md)) and start the server.
4. Run a test import that writes nothing:

    ```
    /dp admin import bigdoors --dry-run
    ```

    You see **Dry run: simulating the import from &lt;file&gt;. Nothing will be written.**, then a summary such as
    **Dry run complete: 12 doors would be imported, 1 skipped, 2 to check, 12 triggers and 5 access entries would be
    created.**

5. Read the report (see [Reading the report](#reading-the-report)). If doors are skipped or need checking, look at
   why before going on.
6. Run the real import:

    ```
    /dp admin import bigdoors
    ```

    You see **Importing doors from the BigDoors database &lt;file&gt;...**, then **Import complete: ...** with the
    same counters.

7. Fix the doors listed as "to check" (see [Doors to check](#doors-to-check)).
8. Stop the server, delete the BigDoors jar from `plugins/`, and start the server again. You can delete the
   `plugins/BigDoors/` folder once you are happy with the result.

!!! tip
    You can also run the command from the server console. Only one import can run at a time: a second one is
    refused with **An import is already running. Wait for it to finish.**

## The command

```
/dp admin import bigdoors [file] [--dry-run]
```

| Part | Meaning |
|---|---|
| `bigdoors` | Required. The only import source. |
| `[file]` | Optional. Path of the BigDoors database. Without it Door+ reads `plugins/BigDoors/doorDB.db`. |
| `[--dry-run]` | Optional. Does everything (reading, mapping, report) except saving the doors. |

`file` and `--dry-run` can be given in any order. How a custom `file` is found:

- An absolute path is used as it is, e.g. `/dp admin import bigdoors /backups/doorDB.db`.
- A relative path is first looked up from the server folder. If nothing exists there, Door+ looks inside
  `plugins/`. So `/dp admin import bigdoors BigDoors/old.db` finds `plugins/BigDoors/old.db`.
- The path cannot contain spaces (each space starts a new command argument) and can be at most 260 characters.

If the file does not exist you see **BigDoors database not found: &lt;file&gt;.**

## Reading the report

The report goes to two places.

**In chat (or console)**: the summary line, then up to 15 detail lines, skipped doors first, then doors to check:

- **Skipped BigDoors door #&lt;id&gt; (&lt;name&gt;): &lt;reason&gt;.**
- **To check: &lt;name&gt; (BigDoors #&lt;id&gt;): &lt;reason&gt;. Fix it with /door set &lt;name&gt; direction or
  base-state.**

When there are more lines you see **...and &lt;count&gt; more. See the report file.** The last line gives the file:
**Full report: &lt;file&gt;.**

**In the log file** `plugins/DoorPlus/import-bigdoors-<yyyy-MM-dd>.log` (date of the import). Every run of that day,
dry runs included, is appended to the same file. A run looks like this:

```
=== BigDoors import 2026-11-15T14:03:27.512 ===
Source: plugins/BigDoors/doorDB.db
Imported: 2 | Skipped: 1 | To check: 1 | Triggers created: 2 | Access entries created: 1
[IMPORTED] bigdoors:1 -> #12 castle_gate (BIG_DOOR, owner Steve 0f3c1a2b-..., world 5b1e7d90-..., base [100,64,200 -> 100,70,205], state 0, direction CW)
[TRIGGER] castle_gate: REDSTONE FOLLOW at 98,64,199
[ACCESS] castle_gate: PLAYER 8d2e4f11-... ADMIN
[IMPORTED] bigdoors:2 -> #13 bridge (DRAWBRIDGE, owner Alex 3a9b..., world 5b1e7d90-..., base [120,64,210 -> 124,64,216], state 0, direction NORTH, pivot axis X)
[TO CHECK] bigdoors:2 bridge: POSITION_MISMATCH
[TRIGGER] bridge: REDSTONE FOLLOW at 122,63,217
[SKIPPED] bigdoors:3 'test': NO_OWNER (no creator in sqlUnion)
```

| Line | Content |
|---|---|
| `[IMPORTED]` | BigDoors id, the new Door+ id (`#12`; `-` in a dry run), name, type, owner, world, closed position (`base`), state (`0` closed, `1` open), opening direction and, for drawbridges, the pivot axis. |
| `[TO CHECK]` | A door that was imported but needs your attention, with the reason code. |
| `[TRIGGER]` | The redstone trigger created from the BigDoors power block. |
| `[ACCESS]` | A co-owner added to the door, with its role. |
| `[SKIPPED]` | A door that was not imported, with the reason code and a technical detail. |

!!! tip
    Use the Door+ id from the `[IMPORTED]` line (e.g. `#12`) in commands. It always points at the right door,
    even if several players own a door with the same name.

## What is mapped

| BigDoors | Door+ |
|---|---|
| Type Door | `big_door` (see [Big door](../doors/types/big-door.md)) |
| Type Drawbridge | `drawbridge` (see [Drawbridge](../doors/types/drawbridge.md)) |
| Type Portcullis | `portcullis` (see [Portcullis](../doors/types/portcullis.md)) |
| Type Sliding door | `sliding_door` (see [Sliding door](../doors/types/sliding-door.md)) |
| Name | Same name. Characters other than letters, digits, `_` and `-` become `_`; cut to 32 characters; an empty name becomes `door`. If the owner already has a door with that name, `_2`, `_3`... is added. |
| Creator | Owner of the door. If BigDoors lists several creators, the first one becomes the owner. |
| Co-owner with BigDoors permission level `1` | Access entry with role **Admin**. |
| Co-owner with BigDoors permission level `2` | Access entry with role **User**. |
| — | Privacy **Restricted** if the door has co-owners, otherwise **Private** (see [Privacy and roles](../access/privacy-and-roles.md)). |
| Power block | A **Redstone** trigger on that block with action **Follow state**: powered = open, unpowered = closed. "Allow redstone" is switched on for the door. |
| Engine (rotation point) | Pivot of big doors and drawbridges. |
| Open direction | Clockwise, Counter-clockwise, Up, Down, North, East, South, West, as in BigDoors. |
| No open direction | The default direction of the type, worked out from the blocks around the door. If the world is not loaded, the first direction the type allows. |
| Blocks to move (portcullis, sliding door) | Same value; `0` or less means the type default. |
| Auto close (seconds) | Same value; `0` or less means off. |
| Locked | Locked. |
| Open or closed | Door+ stores the closed position and the state. For an open door, Door+ works out where the closed door is, then checks that the result matches the blocks BigDoors saved. If it does not match, the door is imported as it stands and listed as "to check". |

Drawbridges get a pivot axis: the horizontal axis of an upright bridge, otherwise the axis given by the BigDoors
engine side. If neither is known, the axis is deduced from the shape of the bridge and the door is listed as
"to check".

Other settings of an imported door take Door+ values: **Protect blocks** follows `defaults.protect-blocks`; speed
×1, no key, region access off, default collisions.

BigDoors player names are added to the Door+ name list, so you can type co-owners' names in commands even if they
never joined since Door+ was installed. A name that is not a valid Minecraft name is replaced by the first 8
characters of the player's UUID.

## What is skipped and why

| Reason in chat | Code in the log | Cause |
|---|---|---|
| already imported | `ALREADY_IMPORTED` | This BigDoors door was imported before (see [Running it again](#running-it-again)). |
| door type not supported | `UNKNOWN_TYPE` | The BigDoors type is not one of the four above. |
| the door has no creator | `NO_OWNER` | BigDoors has no creator with a valid UUID for this door. |
| invalid data | `INVALID_DATA` | The world id is not valid, or the row has data Door+ cannot read. The log shows the detail. |

BigDoors options without a Door+ equivalent are ignored: notifications and "bypass protections".

## Doors to check

A door "to check" is imported and works, but its position or rotation may not match what you had in BigDoors.

| Reason in chat | Code in the log | What happened |
|---|---|---|
| the computed position does not match the BigDoors one, imported closed in its current position | `POSITION_MISMATCH` | Door+ could not reproduce the BigDoors position. The blocks as they stand now are the **closed** position. |
| open door without an opening direction, imported closed in its current position | `OPEN_WITHOUT_DIRECTION` | The door was open in BigDoors but had no direction, so Door+ cannot know where it closes. The blocks as they stand now are the **closed** position. |
| the rotation axis of the drawbridge was deduced from its shape | `PIVOT_AXIS_GUESSED` | The drawbridge may rotate around the wrong axis. |

Test each door with `/dp toggle <door>` and fix it as follows. These commands need the Admin role on the door, or
the bypass permissions an operator has (see [Permissions](../reference/permissions.md)).

**The door opens the wrong way.** Change the direction while the door is closed:

```
/dp set #12 direction <direction>
```

Tab completion lists the directions the door allows (`cw`, `ccw`, `up`, `down`, `north`, `east`, `south`, `west`
depending on the type).

**The door was open in BigDoors, and Door+ now treats that open position as closed.** Move it to its real closed
position, then tell Door+ that this is the closed position:

1. Run `/dp set #12 direction <direction>`, choosing the direction that moves the door towards where it should be
   closed.
2. Run `/dp toggle #12`. The door moves to its real closed position (Door+ thinks it is now open).
3. Run `/dp set #12 base-state`. You see **The current position of &lt;door&gt; is now its closed state.** Door+
   also reverses the direction, so the next opening goes back the way it came.
4. Run `/dp toggle #12` to check that the door opens correctly.

**The drawbridge rotates around the wrong axis.** Redefine it with `/dp redefine #12` and select the pivot again
(see [Editing and deleting](../doors/editing.md)).

## Running it again

Running the import again is safe. Every imported door remembers which BigDoors door it came from, and that door is
skipped as **already imported**. Only BigDoors doors that were not imported yet are added.

If you delete an imported door in Door+ and run the import again, that door is imported again.

If saving fails, nothing is imported (**The import failed while saving: nothing was imported. Details are in the
server console.**). Fix the cause and run the command again.

## Limits are not applied

Imported doors skip every check a player goes through when creating a door: the limits in `limits.*` (number of
doors, door size, triggers, blocks to move, auto-close seconds, access entries, trigger distance),
`worlds.disabled`, `types.<type>.enabled` and protection plugins such as WorldGuard. The import does not check
for overlaps with doors you already created in Door+ either.

!!! note
    Imported doors count towards their owner's door limit. A player who already owns more doors than
    `limits.doors-per-player` keeps all imported doors, but cannot create new ones until they are under the limit.
    See [Limits](../reference/limits.md).

## Common messages

| Message | What to do |
|---|---|
| **Unsupported BigDoors database version &lt;version&gt; (expected 9). Start the latest BigDoors once so it updates its database, then retry.** | Install the latest BigDoors v1, start the server once, then retry. |
| **Cannot read the BigDoors database. Details are in the server console.** | The file is not a valid BigDoors SQLite database, or it is damaged. Check the console. |
| **Cannot write the report file. Details are in the server console.** | The doors were imported (or simulated), only the log file could not be written. Check the folder permissions of `plugins/DoorPlus/`. |
| **&lt;door&gt; cannot move: the block at x, y, z cannot be animated.** (when toggling) | The door contains a block Door+ does not animate. See [Movement and animation](../doors/movement.md). |
