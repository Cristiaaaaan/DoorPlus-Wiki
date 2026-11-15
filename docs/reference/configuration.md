# Configuration

This page explains every key of `plugins/DoorPlus/config.yml`, section by section: type, default, allowed values, effect, and when a change takes effect. It is for server administrators.

## How the file works

- Door+ creates `config.yml` on the first start. After an update, keys that are missing from your file are added automatically with their default value and comment; your existing values are kept.
- Do not edit `config-version`. Door+ uses it to track the file format.
- Durations in **ticks**: 20 ticks = 1 second.
- Values such as `PRIVATE`, `block_display` or `EASE_IN_OUT` are not case-sensitive.
- Material and entity names are the Bukkit names, for example `TRIPWIRE_HOOK`, `ARMOR_STAND`.

### Invalid values

An invalid value never stops the server. Door+ writes a warning to the console such as
`Invalid value for 'limits.max-blocks-to-move': 9999; using default 64` and uses the default value instead. Your file is not changed.

- A number outside its range, a decimal where a whole number is required, or text where a number or `true`/`false` is required counts as invalid.
- Unknown names inside a list (materials, entity types, collision modes) are skipped one by one with a warning; the rest of the list is used. This lets one file work on several Minecraft versions.

### Reload

`/dp admin reload` re-reads `config.yml` and the language files. If the file cannot be read (for example a YAML syntax error), you see **Reload failed: the previous configuration is still active.** and the details are in the console.

| Change | When it takes effect |
|---|---|
| `storage.*` | Only after a server restart. Reload prints `Changes to the 'storage' section require a server restart`. |
| `integrations.*` | Only after a server restart: integrations are hooked when Door+ starts. |
| Everything else | After `/dp admin reload`, from the next action that uses it (next door movement, next creation, next trigger link). |

## General

| Key | Type | Default | Effect |
|---|---|---|---|
| `config-version` | number | `1` | Format version of the file. Do not change it. |
| `debug` | `true`/`false` | `false` | Writes extra diagnostic messages to the console. Turn it on only when you investigate a problem. |

## language

| Key | Type | Default | Effect |
|---|---|---|---|
| `language.default` | text | `en` | Language file used for the console and for players when `per-player` does not apply. Must match a file in `plugins/DoorPlus/lang/` without `.yml` (bundled: `en`, `it`). If no such file exists, Door+ warns and uses `en`. |
| `language.per-player` | `true`/`false` | `true` | Each player sees messages in their Minecraft client language when a matching file exists. |

Details on the [Language](language.md) page.

## storage

Needs a **restart**. Full setup guide on the [Storage](storage.md) page.

| Key | Type | Default | Allowed values | Effect |
|---|---|---|---|---|
| `storage.type` | text | `sqlite` | `sqlite`, `mysql`, `mariadb` | Database used. |
| `storage.server-id` | text | `default` | 1 to 32 characters | Identifies this server when several servers share one database. |
| `storage.table-prefix` | text | `dp_` | 1 to 16 letters, digits or `_` | Prefix of every table name. |
| `storage.sqlite.file` | text | `doorplus.db` | file name | SQLite file, inside `plugins/DoorPlus/`. |
| `storage.mysql.host` | text | `localhost` | | Database server address. |
| `storage.mysql.port` | number | `3306` | `1` to `65535` | Database server port. |
| `storage.mysql.database` | text | `doorplus` | | Database (schema) name. |
| `storage.mysql.username` | text | `doorplus` | | Login user. |
| `storage.mysql.password` | text | `''` | | Login password. |
| `storage.mysql.properties` | text | `'useSSL=false'` | | Extra JDBC connection options, `key=value` joined with `&`. |
| `storage.pool.max-size` | number | `4` | `1` to `64` | Maximum open connections (MySQL/MariaDB only; SQLite always uses 1). |
| `storage.pool.connection-timeout-ms` | number | `10000` | `250` to `600000` | How long to wait for a connection, in milliseconds. |

## worlds

| Key | Type | Default | Effect |
|---|---|---|---|
| `worlds.disabled` | list of world names | `[]` | Doors cannot be created in these worlds. Existing doors there keep working. Write the names exactly as the world folders, for example `[world_nether, minigames]`. |

## defaults

Initial settings of every **new** door. Changing them does not change existing doors.

| Key | Type | Default | Allowed values | Effect |
|---|---|---|---|---|
| `defaults.privacy` | text | `PRIVATE` | `PRIVATE`, `RESTRICTED`, `PUBLIC` | Who can use a new door. See [Privacy and roles](../access/privacy-and-roles.md). |
| `defaults.allow-redstone` | `true`/`false` | `false` | | Whether redstone can operate a new door. |
| `defaults.protect-blocks` | `true`/`false` | `false` | | Whether the blocks of a new door are protected from breaking. |
| `defaults.auto-close-seconds` | number | `0` | `0` to `86400` | Auto-close time of a new door; `0` = off. |
| `defaults.region-access` | `true`/`false` | `false` | | Members and owners of the WorldGuard region around the door can use it (only for `RESTRICTED` doors). See [WorldGuard](../integrations/worldguard.md). |

## limits

Every key of this section, with ranges, permission overrides and examples, is explained on the [Limits](limits.md) page.

| Key | Default |
|---|---|
| `limits.doors-per-player` | `10` |
| `limits.max-door-blocks` | `500` |
| `limits.max-triggers-per-door` | `16` |
| `limits.max-blocks-to-move` | `64` |
| `limits.max-trigger-distance` | `64` |
| `limits.max-access-entries` | `64` |
| `limits.max-proximity-radius` | `16` |
| `limits.max-auto-close-seconds` | `600` |
| `limits.max-platform-stops` | `8` |
| `limits.max-groups-per-player` | `10` |
| `limits.max-doors-per-group` | `16` |
| `limits.speed-multiplier.min` / `.max` | `0.25` / `4.0` |

## creation

| Key | Type | Default | Allowed values | Effect |
|---|---|---|---|---|
| `creation.timeout-seconds` | number | `120` | `20` to `3600` | A door creation with no activity for this long is cancelled. |

## blocks

Which blocks can be part of a door. All values are lists of block materials.

| Key | Default | Effect |
|---|---|---|
| `blocks.replaceable` | `[WATER, LAVA, SNOW, FERN, SHORT_GRASS, TALL_GRASS, SEAGRASS, TALL_SEAGRASS]` | Blocks a moving door may overwrite, and that do not count as door blocks when you select a region. Air is always replaceable. |
| `blocks.passable` | `[BARRIER, LIGHT, STRUCTURE_VOID]` | Blocks a moving door may pass through: the door replaces them while it is there and puts them back when it leaves. Blocks with a block entity (chests, signs, ...) are refused here with a warning. A block listed here is removed from `replaceable`. |
| `blocks.blacklist` | `[BEDROCK, END_PORTAL, END_PORTAL_FRAME, NETHER_PORTAL, END_GATEWAY, COMMAND_BLOCK, CHAIN_COMMAND_BLOCK, REPEATING_COMMAND_BLOCK, STRUCTURE_BLOCK, JIGSAW, REINFORCED_DEEPSLATE]` | Blocks that can never be part of a door. |
| `blocks.whitelist` | `[]` | If not empty, **only** these blocks can be part of a door. |
| `blocks.allow-tile-entities` | `false` | Reserved. Only `false` is accepted in this version: blocks with a block entity (chests, furnaces, signs, ...) cannot be part of a door. `true` is replaced by `false` with a warning. |

If one of the list keys is not a list at all, the default list is used.

## animation

How door movements are drawn.

| Key | Type | Default | Allowed values | Effect |
|---|---|---|---|---|
| `animation.engine` | text | `block_display` | `block_display`, `falling_block` | Entities used to show the moving blocks. `block_display` is smooth and recommended. |
| `animation.update-interval-ticks` | number | `2` | `1` to `5` | How often the animation is updated. Lower is smoother but costs more. |
| `animation.easing` | text | `EASE_IN_OUT` | `LINEAR`, `EASE_IN_OUT` | `EASE_IN_OUT` starts and stops gently; `LINEAR` moves at constant speed. |
| `animation.min-ticks` | number | `10` | `1` to `1200` | Shortest duration of a movement. |
| `animation.max-ticks` | number | `600` | `1` to `12000`, not below `min-ticks` | Longest duration of a movement. |
| `animation.viewer-range` | number | `64` | `1` to `512` | If no player is within this many blocks of the door, it moves instantly without animation. |
| `animation.max-animated-blocks` | number | `1500` | `1` to `1000000` | Doors with more blocks move instantly without animation. |
| `animation.max-global-animated-blocks` | number | `10000` | `1` to `10000000` | Maximum animated blocks on the whole server at once. Movements that would exceed it happen instantly. |
| `animation.display.view-range` | decimal | `1.0` | `0.01` to `64` | View range multiplier of the display entities (`block_display` engine). |
| `animation.display.fixed-brightness` | `true`/`false` | `true` | | Gives moving blocks a fixed brightness so they do not flicker dark while moving. |

## speed

Base speeds. Each door can change them with its speed multiplier (`/dp set <door> speed`). The result is always kept between `animation.min-ticks` and `animation.max-ticks`.

| Key | Type | Default | Allowed values | Effect |
|---|---|---|---|---|
| `speed.blocks-per-second` | decimal | `2.0` | `0.05` to `100` | Speed of sliding movements (sliding doors, portcullises, platforms, ...). |
| `speed.degrees-per-second` | decimal | `45.0` | `1` to `3600` | Speed of rotating movements (big doors, drawbridges, ...). |
| `speed.max-tip-speed` | decimal | `6.0` | `0.1` to `200` | Maximum speed, in blocks per second, of the far edge of a rotating door. Large doors rotate slower so their edge does not exceed it. |

## movement

| Key | Type | Default | Allowed values | Effect |
|---|---|---|---|---|
| `movement.toggle-cooldown-ticks` | number | `0` | `0` to `72000` | Minimum time between two movements of the same door. |
| `movement.check-swept-volume` | `true`/`false` | `true` | | Checks the whole path of the door, not only its end position, for obstacles before moving. |
| `movement.update-neighbors` | `true`/`false` | `true` | | Sends block updates to neighbouring blocks when a door moves (redstone, water and similar react). |
| `movement.auto-close-unloaded` | `true`/`false` | `true` | | When an auto-close is due and the door's chunks are not loaded: `true` closes it instantly; `false` waits until the chunks are loaded. |

## collision

How moving doors treat players and entities. See [Movement](../doors/movement.md).

| Key | Type | Default | Allowed values | Effect |
|---|---|---|---|---|
| `collision.default-mode` | text | `none` | one of `allowed-modes` | Mode of doors set to "default". If it is not in `allowed-modes`, the first allowed mode is used with a warning. |
| `collision.allowed-modes` | list | `[none, shulker, push]` | `none`, `shulker`, `push` | Modes players can choose. `none`: entities pass through; `shulker` (shown as **Solid**): the moving door is solid; `push`: entities are pushed away. An empty list allows all three. |
| `collision.evict-on-finish` | `true`/`false` | `true` | | When a movement ends, entities inside the door's blocks are moved out. |
| `collision.shulker.max-per-door` | number | `200` | `0` to `10000` | Maximum invisible collision entities per door in `shulker` mode. |
| `collision.push.non-players` | `true`/`false` | `true` | | `push` mode also pushes mobs and dropped items, not only players. |
| `collision.push.strength` | decimal | `1.2` | `0` to `10` | Push strength. |
| `collision.push.max-velocity` | decimal | `1.5` | `0.05` to `10` | Maximum speed of a pushed entity. |

## types

| Key | Type | Default | Allowed values | Effect |
|---|---|---|---|---|
| `types.<type>.enabled` | `true`/`false` | `true` | | `false` prevents creating new doors of that type. Existing doors keep working. `<type>` is a type id such as `big_door`. |
| `types.platform.carry-entities` | `true`/`false` | `true` | | Platforms carry the players and entities standing on them. |
| `types.windmill.degrees-per-second` | decimal | `30.0` | `1` to `3600` | Rotation speed of windmills that have no own speed set (`/dp set <door> degrees-per-second`). |
| `types.flag.amplitude` | decimal | `0.6` | `0` to `4` | Height of the flag wave, in blocks. |
| `types.flag.wavelength` | decimal | `6.0` | `0.5` to `128` | Length of one wave, in blocks. |
| `types.flag.period-ticks` | number | `40` | `2` to `1200` | Time for one full wave. |
| `types.clock.timezone` | text | `system` | `system` or a time zone id such as `Europe/Rome`, `UTC` | Time zone of clocks that show real time. `system` uses the server's time zone. An unknown id falls back to `system` with a warning. |

## perpetual

| Key | Type | Default | Allowed values | Effect |
|---|---|---|---|---|
| `perpetual.check-interval-ticks` | number | `20` | `1` to `1200` | How often Door+ checks running always-moving doors (windmill, flag, clock) to show them when a player is near and their chunks are loaded, and to put them to sleep otherwise. |

## triggers

| Key | Type | Default | Allowed values | Effect |
|---|---|---|---|---|
| `triggers.default-cooldown-ticks` | number | `10` | `0` to `72000` | Cooldown given to new triggers. Each trigger can change it. |
| `triggers.selection-timeout-seconds` | number | `30` | `5` to `600` | Time to click a block or entity after starting a trigger selection. |
| `triggers.redstone.enabled` | `true`/`false` | `true` | | `false` turns off redstone on the whole server: redstone triggers and the redstone option of doors stop working. |
| `triggers.entity.allowed-types` | list of entity types | `[ITEM_FRAME, GLOW_ITEM_FRAME, ARMOR_STAND, INTERACTION, PAINTING]` | entity type names except `PLAYER` | Entities that can be linked as entity triggers. |
| `triggers.proximity.check-interval-ticks` | number | `10` | `1` to `200` | How often proximity triggers look for players. |
| `triggers.proximity.ignore-vanished` | `true`/`false` | `true` | | Vanished players do not activate proximity triggers. |

## keys

| Key | Type | Default | Allowed values | Effect |
|---|---|---|---|---|
| `keys.check` | text | `HAND` | `HAND`, `INVENTORY` | Where Door+ looks for the key: only in the hand used, or anywhere in the inventory. See [Keys](../access/keys.md). |
| `keys.default-item` | material | `TRIPWIRE_HOOK` | any item | Item used for key copies (`/dp key give` and the menu). An unknown material, a block-only material or air falls back to `TRIPWIRE_HOOK`. |

## integrations

Needs a **restart**. `false` means Door+ does not hook into that plugin even if it is installed.

| Key | Default | Plugin |
|---|---|---|
| `integrations.luckperms` | `true` | LuckPerms |
| `integrations.placeholderapi` | `true` | PlaceholderAPI |
| `integrations.worldguard` | `true` | WorldGuard |
| `integrations.itemsadder` | `true` | ItemsAdder |
| `integrations.nexo` | `true` | Nexo |
| `integrations.oraxen` | `true` | Oraxen |
| `integrations.craftengine` | `true` | CraftEngine |
| `integrations.citizens` | `true` | Citizens |
| `integrations.fancynpcs` | `true` | FancyNpcs |

See [Integrations](../integrations/index.md).

## placeholders

| Key | Type | Default | Allowed values | Effect |
|---|---|---|---|---|
| `placeholders.nearest-radius` | number | `16` | `1` to `256` | Search radius, in blocks, of the "nearest door" placeholders. |
| `placeholders.looking-distance` | number | `10` | `1` to `128` | Maximum distance, in blocks, of the "door you are looking at" placeholders. |

See [Placeholders](placeholders.md).

## gui

| Key | Default | Effect |
|---|---|---|
| `gui.click-sound` | `{ sound: 'minecraft:ui.button.click', volume: 0.5, pitch: 1.0 }` | Sound played when a menu button is clicked. Set `sound: ''` for no sound. Same fields as the door sounds below. |

## sounds

| Key | Type | Default | Effect |
|---|---|---|---|
| `sounds.enabled` | `true`/`false` | `true` | `false` silences every door sound. |
| `sounds.<type>.start` | sound | depends on type | Played when the door starts moving. |
| `sounds.<type>.loop` | sound | depends on type | Repeated while the door moves, every `interval-ticks`. |
| `sounds.<type>.end` | sound | depends on type | Played when the door stops. |

Each sound has these fields:

| Field | Default | Allowed values | Effect |
|---|---|---|---|
| `sound` | | a vanilla sound key, for example `'minecraft:block.piston.extend'`; `''` = no sound | Sound to play. Only vanilla sounds are supported. The name is not checked: a misspelled name is simply not heard. |
| `volume` | `1.0` | `0` or more | Volume. Larger values are heard from farther away. |
| `pitch` | `1.0` | `0.5` to `2.0` (values outside are clamped) | Pitch. |
| `interval-ticks` | `10` | `1` or more | Loop sounds only: time between repetitions. |

Example, a quieter drawbridge without the loop sound:

```yaml
sounds:
  drawbridge:
    start: { sound: 'minecraft:block.chain.break', volume: 0.6, pitch: 0.6 }
    loop:  { sound: '', volume: 0.0, pitch: 1.0, interval-ticks: 8 }
    end:   { sound: 'minecraft:block.wood.place', volume: 0.6, pitch: 0.5 }
```

The default sounds of every type are in the `config.yml` generated on first start.
