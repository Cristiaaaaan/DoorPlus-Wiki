# Door types

Door+ has 13 door types. This page compares them and helps you pick the right one; each type has its own page with
build rules, creation steps, options and tips. It is for everyone who creates doors.

## All types at a glance

The **id** is what you type in `/dp create <type> [name]`. Ids are not case-sensitive and a dash works as well as an
underscore (`big-door` = `big_door`). Tab completion only lists the types you are allowed to create and that are
enabled on the server.

| Type | Id | What it does | Shape required | Kind |
|---|---|---|---|---|
| [Big door](big-door.md) | `big_door` | Swings 90° around a vertical hinge, like a normal door | Any box | Open / closed |
| [Drawbridge](drawbridge.md) | `drawbridge` | Tips 90° around a horizontal hinge row: a wall falls flat, or a floor rises to a wall | Any box | Open / closed |
| [Portcullis](portcullis.md) | `portcullis` | Slides straight up or down | Any box | Open / closed |
| [Sliding door](sliding-door.md) | `sliding_door` | Slides sideways (north, east, south or west) | Any box | Open / closed |
| [Garage door](garage-door.md) | `garage_door` | Slides up and then folds back under the ceiling (up-and-over) | Vertical wall, 1 block thick | Open / closed |
| [Split door](split-door.md) | `split_door` | Splits into 2 halves or 4 quarters that slide apart | Flat, 1 block thick (wall or floor) | Open / closed |
| [Folding door](folding-door.md) | `folding_door` | Folds in two like a book while it swings open | Vertical wall, 1 block thick, at least 2 wide | Open / closed |
| [Rolling shutter](rolling-shutter.md) | `rolling_shutter` | Rolls up into its top (or any) edge; the rolled-up rows are stored, so it needs no free space | Flat, 1 block thick (wall or floor) | Open / closed |
| [Platform / elevator](platform.md) | `platform` | Moves between 2 to 8 stops in one direction and carries players | Any box | Stops |
| [Revolving door](revolving-door.md) | `revolving_door` | Turns 90° (or 180°, 270°) around its centre every time it is used | Square footprint (width = depth) | Turns on each use |
| [Windmill](windmill.md) | `windmill` | Spins continuously while active | Flat, 1 block thick, square face | Perpetual |
| [Flag](flag.md) | `flag` | Waves continuously while active | Vertical wall, 1 block thick, at least 2 wide | Perpetual |
| [Clock](clock.md) | `clock` | Shows the time with a minute hand and an hour hand | 2 blocks thick, square vertical face | Perpetual |

What the kinds mean:

- **Open / closed**: two positions. The door is built in its closed position; opening moves it to the open position,
  closing moves it back.
- **Stops**: the platform has a list of positions (stop 0, 1, 2, ...) and moves between them.
- **Turns on each use**: the revolving door has no open position. Each use turns it and it always counts as closed.
- **Perpetual**: the windmill, flag and clock have the states **Active** and **Inactive**. While active, the blocks
  keep moving as long as a player is nearby. See [Perpetual types](#perpetual-types).

## How to choose

| You want... | Use |
|---|---|
| A gate or door that swings open | [Big door](big-door.md); [Folding door](folding-door.md) if there is little room to swing |
| A castle bridge over a moat, or a hatch in a floor | [Drawbridge](drawbridge.md) |
| A gate that goes into the ceiling or into the ground | [Portcullis](portcullis.md) |
| A door that slides into a wall pocket | [Sliding door](sliding-door.md) |
| Two wings that slide apart, or a hatch that opens like an iris | [Split door](split-door.md) |
| A garage door that ends up under the ceiling | [Garage door](garage-door.md) |
| A shop shutter, or a door where there is no space at all for the open position | [Rolling shutter](rolling-shutter.md) |
| An elevator or a moving floor with several floors | [Platform / elevator](platform.md) |
| An entrance that turns around a central axis | [Revolving door](revolving-door.md) |
| Decorations that move by themselves | [Windmill](windmill.md), [Flag](flag.md), [Clock](clock.md) |

!!! tip
    Every type except the rolling shutter, the revolving door and the three perpetual types needs **free space**
    where the door ends up when it opens. By default air, water, lava, snow, grass and ferns count as free; barriers,
    light blocks and structure voids count as free too and come back when the door leaves. See
    [Movement and animation](../movement.md#free-space).

## Who can create each type

Two things must allow a type before a player can create it:

| Check | Default | Notes |
|---|---|---|
| Permission `doorplus.create.<type>` (for example `doorplus.create.big_door`) | Operators only | `doorplus.create.*` gives every type. See [Permissions](../../reference/permissions.md). |
| Config key `types.<type>.enabled` (for example `types.big_door.enabled`) | `true` for every type | `false` stops new doors of that type. Existing doors of that type keep working. |

When a type is disabled, `/dp create` answers **Doors of type ... are disabled on this server.** and the type disappears
from tab completion.

## Steps during creation

All types start the same way: name (if you did not type it in the command), first corner, second corner. Then come
the steps of the type, then the optional triggers and the confirmation. The full procedure is on
[Creating a door](../creating.md).

| Type | Extra steps after the two corners |
|---|---|
| Big door | Pivot, direction |
| Drawbridge | Pivot, second click on the pivot row (only if the pivot is on a corner), direction |
| Portcullis | Direction |
| Sliding door | Direction |
| Garage door | Direction |
| Split door | Split mode |
| Folding door | Pivot (hinge), direction |
| Rolling shutter | Direction, visible rows |
| Platform | Direction |
| Revolving door | Direction |
| Windmill | Direction |
| Flag | Pivot (pole) |
| Clock | None (the side you stand on when you click the second corner becomes the front) |

The direction step is skipped when the door has only one valid direction.

## Type settings

Some types have extra settings. You change them by command with `/dp set <door> <option> <value>` or in the door menu
under **Type settings**. You need the Admin role on the door.

| Type | Option | Values | Default |
|---|---|---|---|
| Split door | `mode` | `halves_a`, `halves_b`, `quadrants` | `halves_a` |
| Rolling shutter | `visible-rows` | 0 to rows − 1 | 0 |
| Platform | `stops` | Comma-separated distances, e.g. `0,5,10` | `0,<size along the direction>` |
| Revolving door | `quarter-turns` | 1, 2, 3 | 1 |
| Windmill | `degrees-per-second` | 1 to 360 | `types.windmill.degrees-per-second` (30) |
| Clock | `time-source` | `world`, `real` | `world` |

`mode`, `visible-rows` and `stops` change the shape of the movement, so you can only change them while the door is
closed (state 0) and not moving. The other three can be changed at any time.

Settings shared by all types (direction, blocks to move, speed, auto-close, collisions...) are on
[Editing and deleting](../editing.md).

## Perpetual types

The windmill, flag and clock work differently from the other types:

- Opening a perpetual door makes it **Active**; closing it makes it **Inactive**. Toggle switches between the two.
- While active, the door is only animated when its chunks are loaded and a player is within
  `animation.viewer-range` blocks (default 64). Otherwise its blocks rest in the world in their built position. Door+
  checks this every `perpetual.check-interval-ticks` (default 20 ticks).
- An active door stays active after a server restart and starts moving again when a player comes near.
- Perpetual doors have no collisions: players walk through the moving blocks.

Details are in [Movement and animation](../movement.md#perpetual-types).
