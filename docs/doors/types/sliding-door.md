# Sliding door

The sliding door moves sideways, north, east, south or west, usually into a pocket in the wall next to it. This page
explains how to build one, how far it slides and which direction it picks. Creation id: `sliding_door`.

## What it does

| State | Position |
|---|---|
| Closed (0) | The blocks where you built them |
| Open (1) | The blocks moved horizontally by the number of **blocks to move** |

The blocks only move; they are not turned.

## How to build it

- **Shape**: any box. Usually a wall 1 block thick.
- **Front**: none.
- **Free space**: the cells the door ends up in must be free. When it slides along its own wall, these are usually
  inside the wall next to the doorway, so build a hollow pocket there (air, or barrier blocks; see
  [Passable blocks](../movement.md#passable-blocks)).

**Worked example**: a door 3 blocks wide and 4 blocks tall, in a wall running east–west.

- Direction **East**, default distance: the door slides 3 blocks east (its own width). The 3 × 4 cells east of the
  doorway must be free: build a 3-wide, 4-tall pocket in the wall.
- With `blocks-to-move` set to 2, it slides 2 blocks and one column stays in the doorway.
- Direction **North** (across the wall): the default distance is the door's thickness along that axis, here 1 block,
  so the door steps 1 block out of the wall.

## Creating it

1. Type `/dp create sliding_door [name]`. You receive the creation stick.
2. If you did not give a name, type it in chat.
3. Click the first corner of the door with the stick.
4. Click the opposite corner.
5. **Direction**: click **North**, **East**, **South** or **West**.
6. Add triggers if you want, then click **[Create]** on the summary.

There is no pivot step. The general procedure is on [Creating a door](../creating.md).

## Opening direction

All four horizontal directions are valid. Door+ prefers the directions along the wall:

| Door shape | Default order |
|---|---|
| 1 block thick east–west (a wall running north–south) | North, South, East, West |
| Any other shape | East, West, North, South |

**Default**: the first direction in this order whose end position is free; if none is free, the first one. Change it
later with `/dp set <door> direction <direction>` or the **Direction** button of the door menu, while the door is
closed and not moving.

## Type options

The sliding door has no type settings, but it uses the general **blocks to move** setting:

| | |
|---|---|
| Command | `/dp set <door> blocks-to-move <number>` |
| Menu | **Blocks to move** in the door menu (left +1, right −1, shift ×10) |
| Values | `0` to `limits.max-blocks-to-move` (default 64) |
| Default | `0`, shown as **type default**: the size of the door along the direction (its width when it slides along its wall) |
| When | Only while the door is closed and not moving |

## On the move

- **Speed**: `distance / (speed.blocks-per-second × speed multiplier)` seconds. 3 blocks at the default 2 blocks per
  second take 1.5 seconds.
- **Checks**: the end position and, if `blocks-to-move` is larger than the door's width, the cells between the
  doorway and the end position (see [Free space](../movement.md#free-space)).
- **Entities**: see [Collisions](../movement.md#collisions).

## Sounds

Config section `sounds.sliding_door`:

| Sound | Default |
|---|---|
| `start` | `minecraft:block.piston.extend`, volume 0.8, pitch 0.8 |
| `loop` | none |
| `end` | `minecraft:block.piston.contract`, volume 0.8, pitch 0.8 |

See [Sounds](../movement.md#sounds) for the format.

## Tips and common mistakes

!!! warning "Solid wall where the door should go"
    A sliding door cannot push into a solid wall. If you get **... cannot move: something is in the way at x, y, z.**,
    hollow out the pocket at those coordinates.

!!! tip "Two wings that slide apart"
    Use a [split door](split-door.md) with mode `halves_a` instead of two sliding doors: one door, both wings move
    together.
