# Permissions

This page lists every Door+ permission node, its default, what it unlocks, and a ready-to-copy LuckPerms setup. It is for server administrators.

## How the defaults work

- `true`: every player has it. `op`: only operators have it. `false`: nobody has it unless you grant it (directly or through a parent node).
- **Using** a door or a trigger in the world needs **no permission**: the door's privacy and access entries decide. See [Privacy and roles](../access/privacy-and-roles.md).
- Permissions say what a player may do in general. The **role** on a door (User, Admin, Owner) says what they may do with that door. Commands check both. See [Commands](commands.md).

!!! note "Out of the box"
    With the defaults, a normal player can use and manage the doors on which they have a role, but **cannot create doors**. Grant `doorplus.create.<type>` to the groups that may build. Operators can do everything.

## Player nodes

| Node | Default | What it allows |
|---|---|---|
| `doorplus.user` | `true` | Parent node. Gives all the children below marked "via `doorplus.user`". |
| `doorplus.command.menu` | via `doorplus.user` | Open the menus (`/dp`, `/dp menu`). |
| `doorplus.command.list` | via `doorplus.user` | `/dp list` (own doors). |
| `doorplus.command.info` | via `doorplus.user` | `/dp info`. |
| `doorplus.command.toggle` | via `doorplus.user` | `/dp toggle`, `open`, `close`, `goto`, and `/dp group toggle\|open\|close`. |
| `doorplus.command.manage` | via `doorplus.user` | Manage doors, access entries, triggers, keys and groups by command (`lock`, `rename`, `set`, `privacy`, `transfer`, `delete`, `redefine`, `access`, `trigger`, `key`, `group`). The role on the door is still checked. |
| `doorplus.trigger.*` | via `doorplus.user` | Link triggers of every kind (parent of the nodes below). |
| `doorplus.access.group` | via `doorplus.user` | Add LuckPerms group access entries. |
| `doorplus.access.permission` | via `doorplus.user` | Add permission-node access entries. |
| `doorplus.command.teleport` | `op` | `/dp tp` and the teleport button in the door menu. |

The child nodes themselves (`doorplus.command.menu` and so on) have default `false`: players get them only through `doorplus.user` or by a direct grant. To take one away from everybody, negate it, for example `doorplus.command.manage` = `false`.

## Creating doors

| Node | Default | What it allows |
|---|---|---|
| `doorplus.create.*` | `op` | Create doors of every type (parent of all nodes below). |
| `doorplus.create.big_door` | `op` | Create big doors. |
| `doorplus.create.drawbridge` | `op` | Create drawbridges. |
| `doorplus.create.portcullis` | `op` | Create portcullises. |
| `doorplus.create.sliding_door` | `op` | Create sliding doors. |
| `doorplus.create.garage_door` | `op` | Create garage doors. |
| `doorplus.create.split_door` | `op` | Create split doors. |
| `doorplus.create.folding_door` | `op` | Create folding doors. |
| `doorplus.create.rolling_shutter` | `op` | Create rolling shutters. |
| `doorplus.create.platform` | `op` | Create platforms. |
| `doorplus.create.revolving_door` | `op` | Create revolving doors. |
| `doorplus.create.windmill` | `op` | Create windmills. |
| `doorplus.create.flag` | `op` | Create flags. |
| `doorplus.create.clock` | `op` | Create clocks. |

The pattern is `doorplus.create.<type>` with the type id in lower case. These nodes are checked by `/dp create` and by the **create** button of the main menu, which only shows the types you may create. A type disabled with `types.<type>.enabled: false` cannot be created even with the permission.

## Linking triggers

| Node | Default | What it allows |
|---|---|---|
| `doorplus.trigger.block` | via `doorplus.user` | Link block triggers (buttons, levers, pressure plates, any block). |
| `doorplus.trigger.redstone` | via `doorplus.user` | Link redstone triggers. |
| `doorplus.trigger.custom_block` | via `doorplus.user` | Link custom blocks (ItemsAdder, Nexo, Oraxen, CraftEngine). |
| `doorplus.trigger.furniture` | via `doorplus.user` | Link custom furniture. |
| `doorplus.trigger.entity` | via `doorplus.user` | Link entity triggers (item frames, armor stands, ...). |
| `doorplus.trigger.npc` | via `doorplus.user` | Link NPC triggers (Citizens, FancyNpcs). |
| `doorplus.trigger.proximity` | via `doorplus.user` | Link proximity triggers. |

The pattern is `doorplus.trigger.<kind>`. It is only needed to **link** a trigger; using it needs nothing. See [Triggers](../access/triggers.md).

## Limit nodes

These nodes have no entry in `plugin.yml`: you grant them with the number in the node. They raise or lower a player's limit compared to the configuration. Details are on the [Limits](limits.md) page.

| Node | Overrides | Example |
|---|---|---|
| `doorplus.limit.doors.<n>` | `limits.doors-per-player` | `doorplus.limit.doors.25` |
| `doorplus.limit.size.<n>` | `limits.max-door-blocks` | `doorplus.limit.size.2000` |
| `doorplus.limit.triggers.<n>` | `limits.max-triggers-per-door` | `doorplus.limit.triggers.32` |

The highest `<n>` the player has wins. `-1` means unlimited.

## Admin nodes

| Node | Default | What it allows |
|---|---|---|
| `doorplus.admin.*` | `op` | Every node in this table. |
| `doorplus.admin.list` | `op` | See the doors of every player (`/dp list <player>`, and the matching menu view). |
| `doorplus.admin.bypass.use` | `op` | Use any door, whatever its privacy. Also lets you refer to any door in commands. |
| `doorplus.admin.bypass.edit` | `op` | Act as Admin on any door. |
| `doorplus.admin.bypass.manage` | `op` | Act as Owner on any door, and use other players' groups. |
| `doorplus.admin.bypass.limits` | `op` | Ignore the per-player limits (see [Limits](limits.md) for which ones). |
| `doorplus.admin.bypass.protection` | `op` | Ignore protection plugins (WorldGuard and similar) when creating doors and linking triggers. |
| `doorplus.admin.reload` | `op` | `/dp admin reload` |
| `doorplus.admin.pause` | `op` | `/dp admin pause` |
| `doorplus.admin.stopall` | `op` | `/dp admin stopall` |
| `doorplus.admin.cleanup` | `op` | `/dp admin cleanup` |
| `doorplus.admin.import` | `op` | `/dp admin import` |
| `doorplus.admin.version` | `op` | `/dp admin version` |
| `doorplus.admin.debug` | `op` | `/dp admin debug` |
| `doorplus.admin.claim` | `op` | `/dp admin claim`: take ownership of any door. |

!!! warning "Operators bypass everything"
    Because the bypass nodes default to `op`, an operator can open every private door and is not limited. If your staff are operators but should respect privacy, negate `doorplus.admin.bypass.use` (and the other bypass nodes) for them.

## LuckPerms example setup

This example is for a typical survival server:

- every player can build **big doors** and **sliding doors**, up to 5 doors of 300 blocks each;
- **VIP** players can also build drawbridges, portcullises and garage doors, up to 15 doors of 1000 blocks each and 32 triggers per door;
- **moderators** can see and use every door and teleport to doors;
- **admins** have everything.

Run these commands in the console. In-game, add `/` in front (`/lp ...`). They assume the LuckPerms groups `default`, `vip`, `mod` and `admin` exist, and that `vip` inherits from `default`.

1. Let every player build big doors and sliding doors:

    ```
    lp group default permission set doorplus.create.big_door true
    lp group default permission set doorplus.create.sliding_door true
    ```

2. Set the limits for every player:

    ```
    lp group default permission set doorplus.limit.doors.5 true
    lp group default permission set doorplus.limit.size.300 true
    ```

3. Give VIP more door types:

    ```
    lp group vip permission set doorplus.create.drawbridge true
    lp group vip permission set doorplus.create.portcullis true
    lp group vip permission set doorplus.create.garage_door true
    ```

4. Give VIP higher limits. VIP also inherits the `default` nodes, but the highest number wins, so 15 beats 5:

    ```
    lp group vip permission set doorplus.limit.doors.15 true
    lp group vip permission set doorplus.limit.size.1000 true
    lp group vip permission set doorplus.limit.triggers.32 true
    ```

5. Give moderators read access to every door and teleport:

    ```
    lp group mod permission set doorplus.admin.list true
    lp group mod permission set doorplus.admin.bypass.use true
    lp group mod permission set doorplus.command.teleport true
    ```

6. Give admins everything:

    ```
    lp group admin permission set doorplus.create.* true
    lp group admin permission set doorplus.admin.* true
    lp group admin permission set doorplus.command.teleport true
    ```

7. Check a player's result with `lp user <player> permission check doorplus.create.big_door`.

!!! tip "Limits for everyone without permissions"
    You can also leave out steps 2 and 4 and set `limits.doors-per-player` and `limits.max-door-blocks` in `config.yml`. Limit nodes are only needed where groups should differ from the configuration.

!!! tip "Stop players from managing doors by command"
    To stop a group from managing doors by command, negate `doorplus.command.manage`: `lp group default permission set doorplus.command.manage false`. Using doors in the world needs no permission, so their doors keep working.

See [LuckPerms](../integrations/luckperms.md) for LuckPerms group access entries.
