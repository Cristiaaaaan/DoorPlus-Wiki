# Limits

This page explains every limit Door+ applies: the configuration key, the default, whether a permission can change it, and what counts towards it. It is for server administrators.

All limits live in the `limits` section of `config.yml` and are applied by `/dp admin reload`. For the other configuration keys see [Configuration](configuration.md).

## Overview

| Limit | Config key | Default | Allowed values | Permission override | Ignored with `doorplus.admin.bypass.limits` |
|---|---|---|---|---|---|
| Doors owned per player | `limits.doors-per-player` | `10` | `-1` or more | `doorplus.limit.doors.<n>` | yes |
| Blocks per door | `limits.max-door-blocks` | `500` | `-1` or more | `doorplus.limit.size.<n>` | yes |
| Triggers per door or group | `limits.max-triggers-per-door` | `16` | `-1` or more | `doorplus.limit.triggers.<n>` | yes |
| Groups per player | `limits.max-groups-per-player` | `10` | `-1` or more | none | yes |
| Doors per group | `limits.max-doors-per-group` | `16` | `1` to `1024` | none | yes |
| Blocks to move | `limits.max-blocks-to-move` | `64` | `1` to `4096` | none | no |
| Trigger distance | `limits.max-trigger-distance` | `64` | `0` or more (`0` = unlimited) | none | no |
| Access entries per door | `limits.max-access-entries` | `64` | `1` to `10000` | none | no |
| Proximity radius | `limits.max-proximity-radius` | `16` | `1` to `256` | none | no |
| Auto-close time | `limits.max-auto-close-seconds` | `600` | `0` to `86400` | none | no |
| Platform stops | `limits.max-platform-stops` | `8` | `2` to `64` | none | no |
| Speed multiplier | `limits.speed-multiplier.min` / `.max` | `0.25` / `4.0` | `0.01` to `100`, min not above max | none | no |

`-1` means **unlimited**, and is only accepted where the table says so. A value outside the allowed range is replaced by the default and a warning is written to the console (see [Configuration](configuration.md#invalid-values)).

## What each limit counts

### Doors owned per player

- Counts the doors the player **owns** (Owner role). Doors shared with them do not count.
- Checked when a creation starts and again when it finishes, and when a door is transferred to the player with `/dp transfer`.
- On a transfer to an offline player, Door+ cannot read their permissions, so it uses `limits.doors-per-player`. The console never checks the limit on transfers.
- Message when reached: **You have reached your limit.** (or the creation message with the limit), and for transfers **That player has reached their door limit.**

### Blocks per door

- Counts the blocks in the selected region that are **not** replaceable. Air and the blocks listed in `blocks.replaceable` (water, grass, snow and so on) do not count.
- Checked when a door is created and when its region is selected again with `/dp redefine`.

### Triggers per door or group

- Counts the triggers linked to one target: a door, or a group.
- Checked when a trigger is linked by a player. Triggers added through the API without a player use the config value.

### Groups per player and doors per group

- `limits.max-groups-per-player` counts the groups a player owns. Checked by `/dp group create` and the menu.
- `limits.max-doors-per-group` counts the doors in one group. Checked when a door is added. The bypass only works for a player with `doorplus.admin.bypass.limits`; the console is always limited.

### Other limits

- `limits.max-blocks-to-move`: highest value for the `blocks-to-move` setting of a door.
- `limits.max-trigger-distance`: the farthest a trigger may be from the door's region, in blocks. A trigger must always be in the same world as the door. Only checked when a player links the trigger.
- `limits.max-access-entries`: how many access entries one door can have. The same number limits the own entries of one trigger.
- `limits.max-proximity-radius`: the largest radius of a proximity trigger.
- `limits.max-auto-close-seconds`: the longest auto-close time a door can be given. Doors that already have a longer time close after this maximum. `0` turns auto-close off for every door.
- `limits.max-platform-stops`: the most stops a platform can have.
- `limits.speed-multiplier`: the slowest and fastest speed multiplier a door can be given with `/dp set <door> speed` or the menu.

## Permission overrides

Three limits can be set per player or per group with a permission node that contains the number:

| Node | Replaces |
|---|---|
| `doorplus.limit.doors.<n>` | `limits.doors-per-player` |
| `doorplus.limit.size.<n>` | `limits.max-door-blocks` |
| `doorplus.limit.triggers.<n>` | `limits.max-triggers-per-door` |

How Door+ picks the value, in order:

1. If the player has `doorplus.admin.bypass.limits`, the limit is unlimited.
2. Otherwise, if the player has one or more matching nodes set to `true`, the **highest** `<n>` wins. If any of them is `-1`, the limit is unlimited.
3. Otherwise the config value is used.

!!! warning "A node replaces the config value, also downwards"
    If a player has `doorplus.limit.doors.3` and `limits.doors-per-player` is `10`, the player's limit is **3**. The node is not added to the config value and is not compared with it.

Example: a player in the groups `default` (`doorplus.limit.doors.5`) and `vip` (`doorplus.limit.doors.15`) can own 15 doors. A ready-to-copy LuckPerms setup is on the [Permissions](permissions.md#luckperms-example-setup) page.

Nodes with something other than a whole number after the prefix (for example `doorplus.limit.doors.many`) are ignored.

## Lowering a limit later

Lowering a limit never deletes or changes anything that already exists:

- A player with 12 doors and a new limit of 10 keeps all 12 doors, but cannot create or receive a new one until they own fewer than 10.
- Doors bigger than a new `limits.max-door-blocks` keep working. The limit applies again only if the region is selected again.
- Doors and groups with more triggers, access entries or doors than the new limit keep them; only new additions are refused.

The exception is `limits.max-auto-close-seconds`: doors with a longer auto-close time close after the new maximum.
