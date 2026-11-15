# Rolling shutter

The rolling shutter rolls up into one of its edges, like a shop shutter. The rows that roll past the edge disappear
and are stored by Door+, so the shutter needs **no free space** to open. This page explains how it works, the visible
rows option and what to watch out for. Creation id: `rolling_shutter`.

## What it does

| State | Position |
|---|---|
| Closed (0) | The blocks where you built them |
| Open (1) | Only the **visible rows** remain, next to the roll edge, inside the shutter's own area. All other rows are stored |

The **roll edge** is the edge of the shutter on the side of the direction (the top edge for direction **Up**).

While opening, the whole shutter slides towards the roll edge. Each row that passes the edge shrinks until it vanishes
(with the `falling_block` animation engine it simply disappears at the edge). Closing does the opposite: the stored
rows come back out of the edge and the shutter slides down into place.

The stored rows are kept in the Door+ database, so they survive restarts and crashes.

## How to build it

- **Shape**: flat, 1 block thick: a wall or a floor.
- **Front**: none; the direction decides which edge it rolls into.
- **Free space**: none needed when opening. When closing, the doorway must be empty again.

**Worked example**: a shutter 5 blocks wide and 4 blocks tall, direction **Up**.

- `visible-rows` 0 (default): open, the whole shutter is stored and the doorway is completely empty.
- `visible-rows` 1: the shutter slides up 3 blocks. The 3 top rows roll into the edge and are stored; the **bottom
  row** of the shutter ends up in the top row of the doorway and stays visible there, like the bar of a real
  shutter.

## Creating it

1. Type `/dp create rolling_shutter [name]`. You receive the creation stick.
2. If you did not give a name, type it in chat.
3. Click the first corner of the door with the stick.
4. Click the opposite corner.
5. **Direction**: click the edge to roll into, for example **Up**.
6. **Visible rows**: chat says **Choose Visible rows (current: 0):**. Click a number, **[Other value]** to type one,
   or **[Continue]** to keep 0.
7. Add triggers if you want, then click **[Create]** on the summary.

The general procedure is on [Creating a door](../creating.md).

## Opening direction

The direction is always in the plane of the shutter.

| Shutter | Valid directions | Default order |
|---|---|---|
| Wall running east–west | Up, Down, East, West | Up, Down, East, West |
| Wall running north–south | Up, Down, North, South | Up, Down, North, South |
| Floor | North, East, South, West | North, East, South, West |

**Default**: the first direction of the list. (The default normally picks the first direction whose open position is
free; for the rolling shutter the open position is always inside its own area, so it is always the first one.) Change
it later with `/dp set <door> direction <direction>` or the **Direction** button of the door menu, while the door is
closed and not moving.

## Type options

| | |
|---|---|
| Option | `visible-rows` |
| Command | `/dp set <door> visible-rows <number>` |
| Menu | **Type settings** → **Visible rows** (left +1, right −1, shift ×10) |
| Values | `0` to the number of rows − 1. The number of rows is the size of the shutter along the direction (its height for **Up**/**Down**) |
| Default | `0` |
| When | Only while the door is closed and not moving |

**Blocks to move** does not apply to this type.

## On the move

- **Speed**: `(rows − visible rows) / (speed.blocks-per-second × speed multiplier)` seconds. The example above with
  1 visible row takes 1.5 seconds.
- **Checks when closing**: every cell of the shutter's area that is empty while it is open must be free again. If
  someone placed a block in the doorway, the shutter does not close and reports where.
- **Entities**: see [Collisions](../movement.md#collisions).

## Sounds

Config section `sounds.rolling_shutter`:

| Sound | Default |
|---|---|
| `start` | `minecraft:block.iron_trapdoor.open`, volume 1.0, pitch 1.2 |
| `loop` | `minecraft:block.chain.step`, volume 0.6, pitch 1.0, every 4 ticks |
| `end` | `minecraft:block.iron_trapdoor.close`, volume 1.0, pitch 1.2 |

See [Sounds](../movement.md#sounds) for the format.

## Tips and common mistakes

!!! warning "Deleting an open shutter"
    When you delete an open rolling shutter, Door+ first closes it instantly so the stored rows return to the world.
    If the doorway is blocked, the deletion is refused with **The door cannot close: something is in the way.** Clear
    the doorway and delete again.

!!! warning "Set as closed state"
    The **Set as closed state** button (`/dp set <door> base-state`) does not work for rolling shutters: close the
    shutter instead.

!!! tip "No room above the door"
    The rolling shutter is the type to use when there is no space for an open door at all: under a low ceiling,
    between two other doors, or in a wall too thin for a pocket.
