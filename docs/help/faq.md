# FAQ

Short answers to the questions server owners ask most often. Each answer links to the page with the details. If
something does not work as described, see [Troubleshooting](troubleshooting.md).

## Compatibility

### Which servers does Door+ run on?

Paper **1.21.4 or newer**, including the 26.x versions, with one jar for every version. Door+ needs the Java version
your Paper version needs: 21 for 1.21.x, 25 for 26.x. It has been tested on Paper 1.21.4, 1.21.8, 1.21.11, 26.1.1,
26.2 and 26.3. Spigot and Folia are not supported.

### Does it work for Bedrock players (Geyser)?

No. Door+ is made for Java Edition clients. Bedrock players through Geyser are not supported.

### Do I need a resource pack?

No. Doors are made of vanilla blocks, animations use vanilla entities and all sounds are vanilla sounds.

### Does Door+ need other plugins?

No. LuckPerms, WorldGuard, PlaceholderAPI, ItemsAdder, Nexo, Oraxen, CraftEngine, Citizens and FancyNpcs are all
optional. Door+ detects them at start-up and each one can be turned off in `config.yml`
(`integrations.<name>: false`). See [Integrations](../integrations/index.md).

On the first start, Paper downloads two libraries Door+ needs (HikariCP and the MariaDB driver) from Maven
Central, so the server needs internet access at that moment.

## Doors

### Is Door+ heavy on performance?

No. A door that is not moving is just normal blocks and costs nothing. While a door moves:

- Door+ runs one task for that door only and sends position updates every 2 ticks
  (`animation.update-interval-ticks`); the client smooths the motion in between.
- If no player is within 64 blocks (`animation.viewer-range`), the door moves instantly, without entities.
- Doors with more than 1500 blocks (`animation.max-animated-blocks`) move instantly, and the server never animates
  more than 10000 blocks at once (`animation.max-global-animated-blocks`).

See [Movement and animation](../doors/movement.md).

### What happens if the server crashes while a door is moving?

Nothing is lost. Before every movement Door+ writes the full list of block moves to its database (the journal). On
the next start it completes the interrupted movements and logs
`Journal: recovered <n> of <total> interrupted door movements`. Leftover animation entities are removed
automatically. See [Troubleshooting](troubleshooting.md#leftover-animation-entities).

### How big can a door be?

By default a player can create doors of up to **500 blocks** (`limits.max-door-blocks`). You can raise it in the
config (`-1` = unlimited), per player or group with the permission `doorplus.limit.size.<n>`, or remove all limits
for a player with `doorplus.admin.bypass.limits`. Doors bigger than `animation.max-animated-blocks` (1500) still
work but move instantly. See [Limits](../reference/limits.md).

### Can a door contain chests, signs or other blocks with data?

No. Blocks with a block entity (chests, signs, furnaces, hoppers, banners, heads, ...) cannot be part of a door, and
neither can the blocks in `blocks.blacklist` (bedrock, portals, command blocks, ...). Creating such a door shows
**The block &lt;block&gt; at &lt;x&gt;, &lt;y&gt;, &lt;z&gt; cannot be part of a door.** See
[Creating a door](../doors/creating.md).

### Why can't my players create doors?

Check, in this order:

1. **Permission**: creating needs `doorplus.create.<type>` (for example `doorplus.create.big_door`) or
   `doorplus.create.*`. By default only operators have them. See [Permissions](../reference/permissions.md).
2. **Door limit**: by default a player may own 10 doors (`limits.doors-per-player`). The message is
   **You already own the maximum number of doors (&lt;limit&gt;).**
3. **World**: worlds listed in `worlds.disabled` refuse new doors.
4. **Type disabled**: `types.<type>.enabled: false` refuses new doors of that type.
5. **Protection**: WorldGuard (or another protection plugin) requires build rights where the door is.

See [Troubleshooting](troubleshooting.md#players-cannot-create-doors).

### Why can't other players open my door?

New doors are **private** by default (`defaults.privacy: PRIVATE`): only you and your door admins can use them.
Make the door public, or add the players, LuckPerms groups or permissions to its access list. See
[Privacy and roles](../access/privacy-and-roles.md).

### Can I stop every door at once?

Yes. `/dp admin pause` pauses or resumes all new movements (running animations finish). `/dp admin stopall` finishes
every running animation instantly. See [Commands](../reference/commands.md).

## Data and servers

### I use BigDoors. Can I keep my doors?

Yes, from BigDoors v1. Run `/dp admin import bigdoors --dry-run` to see what would be imported, then
`/dp admin import bigdoors` to import. By default Door+ reads `plugins/BigDoors/doorDB.db`. Remove BigDoors
afterwards: two plugins moving the same blocks corrupt them. See [Coming from BigDoors](../getting-started/bigdoors-import.md).

### Can I use MySQL or MariaDB?

Yes. Set `storage.type` to `mysql` or `mariadb` and fill in the `storage.mysql` section. The default is `sqlite`
(file `plugins/DoorPlus/doorplus.db`). Changes to the `storage` section need a server restart. See
[Storage](../reference/storage.md).

### Can several servers share one database?

They can share the database, but not the doors. Give each server its own `storage.server-id`: each server loads
only the doors and groups created on it. Doors are not synchronised between servers. See
[Storage](../reference/storage.md).

### Which languages are available?

English (`en`) and Italian (`it`). `language.default` sets the server language and, with `language.per-player: true`,
each player sees messages in their client language when a matching file exists. You can edit any message in
`plugins/DoorPlus/lang/` or add a file for another language. See [Language files](../reference/language.md).

### Can I change the configuration without restarting?

Yes, with `/dp admin reload`, which reloads `config.yml` and the language files. Only the `storage` and
`integrations` sections need a restart. See [Configuration](../reference/configuration.md).

## Licence and privacy

### Does Door+ need an internet connection?

Yes, to verify your purchase: at start-up and every 12 hours it asks BuiltByBit or Voxel Shop (the marketplace you
bought from) whether your download is licensed. If the marketplace cannot be reached, Door+ keeps working for
**7 days**. It disables itself only if the marketplace refuses the purchase or stays unreachable for 7 days. See
[Licence, verification and privacy](licence-and-privacy.md).

### What happens to my world if Door+ disables itself?

Nothing bad. Only Door+ stops: running animations are finished instantly and every door stays in the world as
normal blocks. Other plugins and your worlds are not touched. See
[If the purchase is refused](licence-and-privacy.md#if-the-purchase-is-refused).

### Can I use Door+ on more than one server?

Yes, on every Minecraft server that you own or administer, with no server limit and no IP lock. You may not share
or redistribute the jar. See [Licence terms](licence-and-privacy.md#licence-terms-summary).

### What does Door+ send to bStats? Can I turn it off?

Anonymous counts: number of doors and triggers, door types, storage type, animation engine and active
integrations, plus the general data bStats collects for every plugin. Turn it off with `enabled: false` in
`plugins/bStats/config.yml`. See [Anonymous statistics](licence-and-privacy.md#anonymous-statistics-bstats).

### Is there an API for developers?

Yes. The API is free (MIT licence) on JitPack: `com.github.Cristiaaaaan:DoorPlus-API:1.0.0`. See
[Developer API](../developers/api.md).
