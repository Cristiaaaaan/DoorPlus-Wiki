# Placeholders

This page lists every PlaceholderAPI placeholder of Door+, with what it shows and an example. It is for server
owners who build scoreboards, holograms, tab lists or menus with PlaceholderAPI.

The placeholders need PlaceholderAPI on the server. Door+ registers them itself: see
[PlaceholderAPI](../integrations/placeholderapi.md) for the setup.

## How placeholders are read

Every placeholder starts with `%doorplus_` and ends with `%`. After `%doorplus_`, upper and lower case do not matter
in the family and field names: `%doorplus_TOTAL%` works like `%doorplus_total%`.

Some placeholders depend on **who is looking** (the "viewer"): the player whose scoreboard, hologram line or menu is
being filled in. Placeholders marked "online viewer" show the unknown text (see below) when there is no online
player, for example in the console or in a global hologram.

### Text for yes/no and for missing values

| Case | What is shown | Language key |
|---|---|---|
| Yes/no values (`is_open`, `redstone`, `can_use`...) | `true` or `false` | `placeholder.true`, `placeholder.false` |
| Door, group or trigger does not exist, field unknown, value not available | nothing (empty text) | `placeholder.unknown` |

You can change these texts in the language files, e.g. to `Yes` / `No` or `-`. With `language.per-player: true`
each player gets the texts of their own language. See [Language files](language.md).

Fields ending in `_display` use the translated names of the language file (for example **Big door**, **Open**,
**Restricted**). The other fields give fixed technical values that are the same in every language, which is
what you want for conditions in other plugins.

## Global placeholders

| Placeholder | Shows | Example |
|---|---|---|
| `%doorplus_total%` | Number of doors on this server | `42` |
| `%doorplus_total_<type>%` | Number of doors of one type. `<type>` is a type id: `big_door`, `drawbridge`, `portcullis`, `sliding_door`, `garage_door`, `split_door`, `folding_door`, `rolling_shutter`, `platform`, `revolving_door`, `windmill`, `flag`, `clock` | `%doorplus_total_drawbridge%` → `5` |
| `%doorplus_moving%` | Doors moving right now | `1` |
| `%doorplus_triggers%` | Number of triggers | `87` |
| `%doorplus_groups%` | Number of door groups | `6` |
| `%doorplus_version%` | Door+ version | `1.0.0` |

## Player placeholders

These describe the viewer.

| Placeholder | Shows | Example |
|---|---|---|
| `%doorplus_player_owned%` | Doors the viewer owns | `3` |
| `%doorplus_player_owned_<type>%` | Doors of one type the viewer owns (type ids as above) | `%doorplus_player_owned_big_door%` → `2` |
| `%doorplus_player_limit%` | The viewer's door limit; `∞` if unlimited (online viewer) | `10` |
| `%doorplus_player_remaining%` | Doors the viewer can still create (never below `0`); `∞` if unlimited (online viewer) | `7` |
| `%doorplus_player_shared%` | Doors the viewer does not own but has a role on (online viewer) | `4` |
| `%doorplus_player_admin_of%` | Doors on which the viewer has exactly the **Admin** role (online viewer) | `1` |
| `%doorplus_player_groups%` | Door groups the viewer owns | `2` |
| `%doorplus_player_creating%` | Whether the viewer is creating a door right now (online viewer) | `false` |

The limit comes from `limits.doors-per-player` or from `doorplus.limit.doors.<n>` permissions (see
[Limits](limits.md)). Roles counted by `player_shared` include roles given by LuckPerms groups, permissions and
WorldGuard region access.

## Door placeholders

The same fields are available in three forms, which differ only in **which door** they describe:

| Form | Door described |
|---|---|
| `%doorplus_door_<ref>_<field>%` | The door named by `<ref>` (see [Door and group references](#door-and-group-references)) |
| `%doorplus_nearest_<field>%` | The door closest to the viewer, within `placeholders.nearest-radius` blocks (online viewer) |
| `%doorplus_looking_<field>%` | The door block the viewer is looking at, within `placeholders.looking-distance` blocks (online viewer) |

If there is no such door (no door within range, not looking at a door, unknown reference), every field shows the
unknown text.

### Fields

The examples describe a big door called `castle_gate`, id `12`, owned by Steve.

| Field | Shows | Example |
|---|---|---|
| `id` | Door id | `12` |
| `name` | Door name | `castle_gate` |
| `type` | Type id | `big_door` |
| `type_display` | Type name from the language file | `Big door` |
| `state` | `open`, `closed` or `moving`; perpetual types (windmill, flag, clock) `active` or `inactive`; platforms `stop_<n>` or `moving`; revolving doors `closed` or `moving` | `closed` |
| `state_display` | State name from the language file (**Open**, **Closed**, **Moving**, **Active**, **Inactive**, **Stop &lt;n&gt;**) | `Closed` |
| `is_open` | Whether the door is open (platforms: not at stop 0; revolving doors: always `false`) | `false` |
| `is_moving` | Whether the door is moving | `false` |
| `is_locked` | Whether the door is locked | `false` |
| `stop` | Current stop of a platform; unknown text for other types | `2` |
| `stops` | Number of stops of a platform; unknown text for other types | `3` |
| `owner` | Owner name | `Steve` |
| `owner_uuid` | Owner UUID | `069a79f4-44e9-4726-a5be-fca90e38aaf5` |
| `privacy` | `private`, `restricted` or `public` | `restricted` |
| `privacy_display` | Privacy name from the language file | `Restricted` |
| `world` | World name | `world` |
| `x`, `y`, `z` | Block at the centre of the door's current position | `100` / `67` / `202` |
| `location` | The same as `x, y, z` | `100, 67, 202` |
| `dimensions` | Size of the closed door, width (X) × height (Y) × depth (Z) | `1x7x6` |
| `blocks` | Number of blocks in the door | `42` |
| `triggers` | Triggers linked directly to the door (triggers of its groups are not counted) | `2` |
| `access_count` | Number of access entries | `3` |
| `auto_close` | Auto-close delay in seconds, `0` = off | `10` |
| `auto_close_remaining` | Seconds left before the door closes by itself, `0` if no auto-close is pending | `7` |
| `speed` | Speed multiplier | `1.5` |
| `redstone` | Whether **Allow redstone** is on | `true` |
| `protected` | Whether **Protect blocks** is on | `false` |
| `key_mode` | `none`, `alternative` or `required` | `none` |
| `toggle_count` | How many times the door has moved | `318` |
| `last_toggled_by` | Last player who moved the door; unknown text if none | `Alex` |
| `last_toggled_ago` | Seconds since that player moved it; unknown text if never | `45` |
| `distance` | Distance from the viewer to the closest block of the door, one decimal; unknown text in another world (online viewer) | `3.5` |
| `role` | Viewer's role: `owner`, `admin`, `user` or `none` (online viewer) | `user` |
| `role_display` | Role name from the language file (online viewer) | `User` |
| `can_use` | Whether the viewer passes the use check of the door: privacy, role, key and bypass permission. Trigger settings and the locked state are not part of it (online viewer) | `true` |

`role` and `can_use` include roles from LuckPerms groups, permission entries and WorldGuard region access. Privacy,
roles and keys are explained in [Privacy and roles](../access/privacy-and-roles.md) and [Keys](../access/keys.md).

Examples:

| Placeholder | Result |
|---|---|
| `%doorplus_door_12_state_display%` | `Closed` |
| `%doorplus_door_Steve:castle_gate_is_open%` | `false` |
| `%doorplus_nearest_name%` | `castle_gate` |
| `%doorplus_looking_owner%` | `Steve` |
| `%doorplus_nearest_distance%` | `3.5` |

## Group placeholders

`%doorplus_group_<ref>_<field>%`, where `<ref>` names a door group in the same way as a door (see
[Door groups](../access/groups.md)).

| Field | Shows | Example |
|---|---|---|
| `name` | Group name | `west_wing` |
| `owner` | Owner name | `Steve` |
| `doors` | Number of doors in the group | `4` |
| `open` | How many of them are open | `1` |
| `closed` | How many are not open | `3` |
| `state` | `all_open`, `all_closed` or `mixed` | `mixed` |

Example: `%doorplus_group_7_state%` → `mixed`.

## Trigger placeholders

`%doorplus_trigger_<id>_<field>%`, where `<id>` is the trigger number shown by `/dp trigger list` (`12` or `#12`).
See [Triggers](../access/triggers.md).

| Field | Shows | Example |
|---|---|---|
| `type` | `block`, `redstone`, `custom_block`, `furniture`, `entity`, `npc` or `proximity` | `redstone` |
| `provider` | `vanilla`, or the plugin that provides the source: `itemsadder`, `nexo`, `oraxen`, `craftengine`, `citizens`, `fancynpcs` | `vanilla` |
| `action` | `toggle`, `open`, `close`, `follow` or `goto` | `follow` |
| `access` | `inherit`, `public`, `restricted` or `owner_only` | `inherit` |
| `target` | Name of the door or group the trigger operates | `castle_gate` |
| `enabled` | Whether the trigger is enabled | `true` |
| `world` | World of the source | `world` |
| `x`, `y`, `z` | Position of the source | `98` / `64` / `199` |
| `location` | The same as `x, y, z` | `98, 64, 199` |

## Door and group references

`<ref>` in `door_<ref>_...` and `group_<ref>_...` can be written in three ways:

| Reference | Example | Finds |
|---|---|---|
| Id | `12` or `#12` | The door (or group) with that id, whoever owns it. |
| `owner:name` | `Steve:castle_gate` | The door (or group) of that player with that name. |
| Name | `castle_gate` | A door (or group) **of the viewer** with that name. |

!!! warning "Use the id for shared displays"
    A bare name only finds the viewer's own doors. In a hologram or scoreboard that many players see, use the id
    (`%doorplus_door_12_state%`) or `owner:name`, otherwise every player who does not own a door with that name
    sees the unknown text. The id is shown by `/dp info <door>` and in the door list.

Placeholders do not check permissions: any player can display the state of any door by its id.

### Names that contain underscores

Door and group names may contain `_`, which is also the separator before the field. Door+ solves this by taking the
**longest field name** that the placeholder ends with, and treating everything before it as the reference.

- `%doorplus_door_castle_gate_type_display%` → field `type_display`, door `castle_gate`.
- `%doorplus_door_old_x_location%` → field `location`, door `old_x`.

If a name ends with something that looks like a field, use the id instead of the name.

## Settings

| Key | Default | Allowed | Effect |
|---|---|---|---|
| `placeholders.nearest-radius` | `16` | 1–256 | How far (in blocks) `nearest_` looks for a door. |
| `placeholders.looking-distance` | `10` | 1–128 | How far (in blocks) `looking_` follows the viewer's line of sight. |

An invalid value is logged and replaced by its default. `/dp admin reload` applies the change. See
[Configuration](configuration.md).

## Caching and performance

All values come from memory: reading a placeholder never touches the database.

Values that need the viewer's position are cached per player, so a scoreboard with many lines does not repeat the
work for each line:

| Value | Recomputed at most every |
|---|---|
| `nearest_` door | 10 ticks (0.5 s) |
| `looking_` door | 5 ticks (0.25 s) |
| `distance`, `can_use` when requested outside the main server thread | 10 ticks (0.5 s) |

Some scoreboard and tab plugins fill in placeholders outside the main server thread. In that case Door+ returns
the last cached value and refreshes it on the next tick, so the very first read for a player can show the unknown
text for a moment. The cache of a player is cleared when they leave.
