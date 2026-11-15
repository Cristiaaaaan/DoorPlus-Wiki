# PlaceholderAPI

This page explains how to use the Door+ placeholders in other plugins through PlaceholderAPI. It is for server
owners who build scoreboards, holograms, tab lists and menus.

## Setup

1. Install PlaceholderAPI.
2. Restart the server.

That is all. The Door+ expansion is built into the plugin: you do **not** need `/papi ecloud download`. At startup the
console shows:

```
PlaceholderAPI <version>: active (expansion doorplus)
```

The expansion is called `doorplus` and stays registered after `/papi reload`.

To test a placeholder in game, use PlaceholderAPI's parse command:

```
/papi parse me %doorplus_version%
```

The full list of placeholders, with examples, is on the [Placeholders](../reference/placeholders.md) page.

To keep Door+ away from PlaceholderAPI, set `integrations.placeholderapi: false` in `config.yml` and restart (see
[Integrations](index.md)).

## Choosing the right placeholder

| You want to show | Use |
|---|---|
| The state of one specific door, to everyone | `%doorplus_door_<id>_<field>%`, e.g. `%doorplus_door_12_state_display%` |
| Information about the door a player is standing next to | `%doorplus_nearest_<field>%` |
| Information about the door a player is looking at | `%doorplus_looking_<field>%` |
| Statistics about the player | `%doorplus_player_<field>%` |
| Statistics about the server | `%doorplus_total%`, `%doorplus_moving%`, ... |

!!! warning "Holograms and shared scoreboards"
    Refer to a door by its **id** (`12` or `#12`) or by `owner:name`. A bare door name only finds doors owned by
    the player viewing the line, so other players would see an empty value. Get the id with `/dp info <door>`.

## Example: scoreboard

Lines for a scoreboard plugin that shows information about the door next to the player:

```
Door: %doorplus_nearest_name%
State: %doorplus_nearest_state_display%
Owner: %doorplus_nearest_owner%
Can use: %doorplus_nearest_can_use%
Your doors: %doorplus_player_owned%/%doorplus_player_limit%
```

When no door is within `placeholders.nearest-radius` blocks (default 16), the door lines show the unknown text,
which is empty by default. Change `placeholder.unknown` in the language file if you prefer a dash or "none" (see
[Language files](../reference/language.md)).

## Example: hologram above a gate

Lines for a hologram plugin, for the door with id 12:

```
%doorplus_door_12_name%
%doorplus_door_12_state_display%
Opened %doorplus_door_12_toggle_count% times
Last used by %doorplus_door_12_last_toggled_by%
```

For a group of doors (id 7):

```
West wing: %doorplus_group_7_open%/%doorplus_group_7_doors% doors open
```

## Yes/no values

Placeholders such as `is_open`, `is_locked` or `can_use` show `true` or `false`. These texts come from the keys
`placeholder.true` and `placeholder.false` of the language file, so you can change them, e.g. to `Yes` / `No`.
Placeholder values are plain text: MiniMessage colour tags in these keys are removed.
