# Troubleshooting

This page is for server owners and staff. Each entry starts from what you see (a console line, a chat message, a
door that does nothing), explains the cause and gives the fix. Chat messages are the default English texts; console
lines are written by Door+ with the `[DoorPlus]` prefix.

!!! tip "Run the action from a command to see the reason"
    Some refusals are silent when they come from a trigger. Run `/dp toggle <door>` (or the same from the console)
    to get the message that explains why the door does not move.

## Door+ is disabled at startup

When Door+ cannot start, it disables itself. Depending on when the error happened, `/dp` is then an unknown
command or answers **Door+ is disabled. Details are in the server console.** In both cases, search the console for
the first Door+ error after `Enabling DoorPlus`.

| Console line | Cause | Fix |
|---|---|---|
| `This copy of Door+ was not downloaded from BuiltByBit or Voxel Shop. ...` | The jar is not an original marketplace download | Download the jar again from your purchases page. See [Licence, verification and privacy](licence-and-privacy.md#if-door-was-disabled-by-mistake) |
| `The marketplace reports that this download of Door+ has no usage rights ...` | The marketplace refused the purchase | See [If Door+ was disabled by mistake](licence-and-privacy.md#if-door-was-disabled-by-mistake) |
| `The purchase of Door+ could not be verified for 7 days (marketplace unreachable) ...` | The server could not reach the marketplace for 7 days | Allow outgoing HTTPS to the marketplace, then restart |
| `Cannot load config.yml or the language files; disabling Door+` | `config.yml` is not valid YAML (wrong indentation, a tab, a missing quote), or the `plugins/DoorPlus/lang/` folder cannot be written | Fix the line named in the error below that message, or rename `config.yml` so that Door+ creates a fresh copy, then restart |
| `Cannot open the database (sqlite): Database schema version <n> is newer than the supported version 2; update Door+ or restore a backup; disabling Door+` | The database was written by a newer Door+ version (you downgraded the plugin) | Install the newer Door+ again, or restore a backup of the database made before the update |
| `Cannot open the database (mysql): Cannot open the mysql database: <details>; disabling Door+` (or `mariadb`) | Door+ cannot connect to the MySQL/MariaDB server | Check `storage.mysql.host`, `port`, `database`, `username` and `password`; check that the database exists, the user has rights on it and the server is reachable from the Minecraft host |
| `Cannot open the database (sqlite): Cannot back up the SQLite database before migrating; disabling Door+` | Before updating the database format Door+ copies `doorplus.db` to `doorplus.db.bak-v<version>`, and the copy failed | Free disk space and check that the server can write to `plugins/DoorPlus/` |
| `Door+ failed to start; disabling` | Unexpected error | Collect the information listed [below](#collecting-information-for-a-bug-report) and report it |

!!! note
    An invalid **value** in `config.yml` (for example a number out of range) does not stop Door+: it is logged and
    replaced by its default. Only a `config.yml` that cannot be read at all stops the start-up. A broken language
    file does not stop Door+ either: the console shows `Missing language key '<key>' in lang/<language>.yml; using
    the bundled text` and the original English or Italian text is used. Changes to the `storage` section need a
    restart; `/dp admin reload` only logs `Changes to the 'storage' section require a server restart`.

If Door+ starts but then disables itself after a few seconds, read the purchase verification messages in
[Licence, verification and privacy](licence-and-privacy.md).

## A door does not move

Find the message you see in the table. Messages marked "action bar" appear above the hotbar instead of in chat.

| Message | Cause | Fix |
|---|---|---|
| **Door movements are paused by an administrator.** | An admin ran `/dp admin pause` | Run `/dp admin pause` again to resume |
| **&lt;door&gt; is locked.** (action bar) | The door is locked; a lock blocks every trigger, command and the API | `/dp unlock <door>` |
| **You cannot use &lt;door&gt;.** (action bar) | The player has no access: privacy, access list, key or trigger access mode | See [Players cannot use a door](#players-cannot-use-a-door) |
| **&lt;door&gt; cannot move: the block at &lt;x&gt;, &lt;y&gt;, &lt;z&gt; cannot be animated.** | A block of the door is not allowed in doors: in `blocks.blacklist`, missing from a non-empty `blocks.whitelist`, or a block with data (chest, sign, ...) | Remove or replace that block, or change the block lists |
| **&lt;door&gt; cannot move: something is in the way at &lt;x&gt;, &lt;y&gt;, &lt;z&gt;.** | A block, or another door, is in the destination or, for rotating doors, in the arc the door sweeps (`movement.check-swept-volume`) | Clear that block. Blocks in `blocks.replaceable` (water, grass, snow, ...) and `blocks.passable` (barrier, light, structure void) never block a door |
| **&lt;door&gt; cannot move beyond the height limits of the world.** | The destination is above the build limit or below the world bottom | Change the direction or the blocks to move (`/dp set <door> direction`, `blocks-to-move`) |
| **&lt;door&gt; cannot move here: denied by &lt;hook&gt;.** | A protection plugin forbids doors in the destination | See [WorldGuard denies an action](#worldguard-denies-an-action) |
| **The world of &lt;door&gt; is not loaded.** (commands only) | The door's world is not loaded | Load the world |
| **Door not found.** (commands only) | The name is wrong, or you have no role on that door | Use `#<id>` or `<owner>:<name>`; see [Players cannot edit a door](#players-cannot-edit-a-door) |
| **&lt;door&gt; could not move because of an error. Details are in the server console.** | Unexpected error | Report it with the console error, see [below](#collecting-information-for-a-bug-report) |

Silent cases: the door does not move and **no message** is shown when:

- the door is **already** in the requested state (an "Open" trigger on an open door);
- the door is **already moving**;
- the door **cooldown** has not expired (`movement.toggle-cooldown-ticks`, 0 by default);
- **another plugin** cancelled the movement through the API.

## A trigger does nothing

1. Run `/dp trigger inspect` and click the source: Door+ lists the triggers linked to it, with action and state.
   **No triggers you can see are linked to this source.** means the trigger is missing or belongs to another
   door.
2. Run `/dp trigger list <door>` and check that the trigger is enabled. Enable it with
   `/dp trigger set <trigger id> enabled true`.
3. Check the cases below.

| Symptom | Cause | Fix |
|---|---|---|
| A redstone trigger never works | Redstone triggers are off on the server (`triggers.redstone.enabled: false`), or the door does not allow redstone. New doors do not allow it by default (`defaults.allow-redstone: false`) | `/dp set <door> redstone true`; ask an admin to set `triggers.redstone.enabled: true` |
| A player clicks and nothing happens, no message | The player was holding an item while sneaking (that click is ignored on purpose), or another plugin denied the click (for example a protection plugin in that area) | Click with an empty hand or without sneaking; allow block interaction in that area |
| The action bar shows **You cannot use &lt;door&gt;.** | The player has no access | Change the door privacy, or the trigger access mode with `/dp trigger set <trigger id> access <mode>` |
| The trigger works only every few clicks | The trigger cooldown (`triggers.default-cooldown-ticks`, 10 ticks by default) or the door is still moving | Wait for the movement to end, or lower the cooldown with `/dp trigger set <trigger id> cooldown <ticks>` |
| An "Open" (or "Close") trigger does nothing | The door is already open (or closed) | Use the "Toggle" action |
| A custom block, furniture or NPC trigger stopped working | The integration is missing or off; the console shows `<Plugin>: not installed; ...` or `<Plugin>: disabled in config (integrations.<name>); ...` | Install the plugin or set `integrations.<name>: true`, then restart |
| The trigger disappeared | Its source (entity, custom block, furniture, NPC) was removed. The door owner sees **The source of trigger #&lt;id&gt; (&lt;type&gt;) of &lt;door&gt; at &lt;x&gt; &lt;y&gt; &lt;z&gt; disappeared: the trigger was removed.** | Link the trigger again |

See [Triggers](../access/triggers.md) for actions, access modes and every trigger setting.

## Players cannot open or edit doors

### Players cannot use a door

The action bar shows **You cannot use &lt;door&gt;.**

1. Check the privacy with `/dp info <door>`. New doors are **Private** by default: only the owner and the admins
   can use them.
2. To let everyone use it: `/dp privacy <door> public`.
3. To let only some players use it: `/dp privacy <door> restricted`, then add them with
   `/dp access add <door> player <name>` (or `group <group>`, `permission <node>`).
4. If the door has a key with mode **Key required**, the player must also hold the key.
5. If the player uses a trigger, the trigger access mode can override the privacy (`/dp trigger list <door>`).

See [Privacy and roles](../access/privacy-and-roles.md) and [Keys](../access/keys.md).

### Players cannot edit a door

| Message | Cause | Fix |
|---|---|---|
| **Requires role &lt;role&gt; on this door.** (menu) | The button needs a higher role: Admin to change settings and triggers, Owner to change privacy, access, transfer or delete | Ask the owner to give the player the Admin role |
| **You need to be an admin of that door.** (command) | Same, from a command | Same |
| **Door not found.** | The player has no role on the door, so it is hidden from them | Add the player to the access list |
| **You do not have permission to do this.** | The player lacks a command permission. By default every player has them through `doorplus.user` | Check that your permission plugin does not remove `doorplus.user` or `doorplus.command.*` |

To take over a door whose owner is gone, an admin can run `/dp admin claim <door>`. See
[Permissions](../reference/permissions.md).

### Players cannot create doors

| Message | Cause | Fix |
|---|---|---|
| **You do not have permission to do this.** | Missing `doorplus.create.<type>`; by default only operators have it | Give `doorplus.create.<type>` or `doorplus.create.*` |
| **You already own the maximum number of doors (&lt;limit&gt;).** | `limits.doors-per-player` (10 by default) | Raise the limit, or give `doorplus.limit.doors.<n>` |
| **The region has more than &lt;limit&gt; blocks, the most you can use for a door.** | `limits.max-door-blocks` (500 by default) | Raise the limit, or give `doorplus.limit.size.<n>` |
| **Doors cannot be created in the world &lt;world&gt;.** | The world is in `worlds.disabled` | Remove it from the list |
| **Doors of type &lt;type&gt; are disabled on this server.** | `types.<type>.enabled: false` | Set it to `true` |
| **You cannot create a door here (denied by &lt;hook&gt;).** | A protection plugin denies the area | See [WorldGuard denies an action](#worldguard-denies-an-action) |
| **You need a free inventory slot for the creation stick.** | Full inventory | Free one slot |

See [Creating a door](../doors/creating.md) and [Limits](../reference/limits.md).

## Animation invisible, choppy or flickering

| Symptom | Cause | Fix |
|---|---|---|
| The door jumps to its new position without animation | No player was within `animation.viewer-range` (64 blocks); or the door has more blocks than `animation.max-animated-blocks` (1500); or the server was already animating `animation.max-global-animated-blocks` (10000); or a chunk of the door had to be loaded | Expected behaviour. Raise these values if you need it |
| The moving blocks disappear when seen from far away | The animation entities have a limited view range | Raise `animation.display.view-range` (1.0 by default) |
| Moving blocks look darker or flicker | Light changes while blocks move | Keep `animation.display.fixed-brightness: true` |
| Console: `animation.engine is falling_block but that engine is unavailable; using block_display` | The `falling_block` engine does not work on this server version | Use `animation.engine: block_display` (the default) |
| Console: `Shulker colliders cannot be spawned (blocked by another plugin?); the shulker collision mode falls back to push` | Another plugin prevents spawning shulkers | Allow Door+ to spawn entities, or use another collision mode |
| The animation is too fast or too slow | Speed settings | Change the door speed (`/dp set <door> speed <multiplier>`) or `speed.*` in the config |

See [Movement and animation](../doors/movement.md).

## Leftover animation entities

Animation entities are never saved with the world, and Door+ removes leftovers by itself: at start-up (the console
shows `Removed <n> orphan animation entities`) and every time a chunk with leftovers loads.

If you still see frozen floating blocks after a crash:

1. Stand near them so that their chunks are loaded.
2. Run `/dp admin cleanup`. The reply is **Removed &lt;count&gt; orphan entities.**

Missing door blocks after a crash are restored from the journal at the next start
(`Journal: recovered <n> of <total> interrupted door movements`). If the console shows
`Journal row of door #<id> kept: its world is not loaded`, load that world and restart the server: the movement is
completed then.

## WorldGuard denies an action

| Message | Cause | Fix |
|---|---|---|
| **You cannot create a door here (denied by WorldGuard).** | The player cannot build in a region the door touches (closed or open position), or a region has the flag `doorplus-create` set to `deny` | Add the player as a member of the region, or set `doorplus-create` to `allow`. Players with `doorplus.admin.bypass.protection` or WorldGuard bypass skip this check |
| **&lt;door&gt; cannot move here: denied by WorldGuard.** | A region where the door would move has `doorplus-toggle` set to `deny`. This applies to everyone, including admins | Remove the flag, or set it to `allow` |
| **The protection WorldGuard does not allow you to link this source.** | The player cannot build where the trigger source is | Add the player to that region, or link the source as someone who can build there |

See [WorldGuard](../integrations/worldguard.md).

## Placeholders are empty

1. Check the console at start-up. You must see `PlaceholderAPI <version>: active (expansion doorplus)`. If you see
   `PlaceholderAPI: not installed; Door+ placeholders unavailable` or `disabled in config`, install PlaceholderAPI or
   set `integrations.placeholderapi: true`, then restart. Door+ registers its own expansion: do not download one
   from the eCloud.
2. Check the spelling: every placeholder starts with `%doorplus_`. An unknown placeholder returns an empty text.
3. Door names in placeholders work like in commands: a plain name means a door **of the player viewing it**. For
   other doors use `#<id>` or `<owner>:<name>`.
4. Player placeholders, and `nearest` / `looking` placeholders, need an online player. `nearest` only finds doors
   within `placeholders.nearest-radius` (16 blocks) and `looking` within `placeholders.looking-distance`
   (10 blocks).
5. The text shown for unknown values is the `placeholder.unknown` key of the language file, empty by default.

See [Placeholders](../reference/placeholders.md).

## Collecting information for a bug report

Send all of the following on the [support Discord](https://discord.gg/bHVeABaWF):

1. Run `/dp admin version` and copy every line: plugin, API and server versions, storage, animation engine,
   integrations, hooks and providers, numbers of doors, triggers and groups.
2. If a specific door misbehaves, run `/dp admin debug <door>`. The reply is
   **State of &lt;door&gt; #&lt;id&gt; written to the console.**; copy the `Debug dump of ...` block from the console.
3. Turn on detailed logs: set `debug: true` in `config.yml` and run `/dp admin reload`. Door+ then writes extra
   lines starting with `[debug]` (triggers linked and removed, auto-close results, creation problems). Reproduce
   the problem, then set `debug: false` again.
4. Attach `logs/latest.log` from the moment the server started. Include full error stack traces, unchanged: Door+
   is obfuscated, and the author can read the scrambled names.
5. Describe the exact steps to reproduce the problem and what you expected.
6. If relevant, attach `config.yml`. Remove `storage.mysql.password` first.
