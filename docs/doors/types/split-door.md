# Split door

The split door divides into two halves or four quarters that slide apart: two wings that open sideways, a wall that
opens top and bottom, or a floor hatch that opens like an iris. This page explains the split modes, where the panels
go and how to build the space for them. Creation id: `split_door`.

## What it does

| State | Position |
|---|---|
| Closed (0) | The blocks where you built them |
| Open (1) | Each panel moved outwards, away from the centre |

The blocks only move; they are not turned. The panels move at the same time.

## How to build it

- **Shape**: flat, 1 block thick: a wall or a floor.
- **Front**: none and no direction: the panels always move outwards.
- **Free space**: each panel needs free cells where it ends up, outside the door's own area. See
  [Free space](../movement.md#free-space).

### Split modes

The door has two axes in its plane. For a wall they are *horizontal* and *vertical*; for a floor they are *east–west*
and *north–south*.

| Mode | Name in the creation chat | Wall | Floor |
|---|---|---|---|
| `halves_a` (default) | **Two halves (side by side)** | Left and right halves slide sideways apart | East half slides east, west half slides west |
| `halves_b` | **Two halves (stacked)** | Top half slides up, bottom half slides down | North half slides north, south half slides south |
| `quadrants` | **Four quadrants (iris)** | Four quarters slide diagonally to the four corners | Four quarters slide diagonally to the four corners |

Rules:

- Every axis that the mode splits needs at least 2 blocks: `halves_a` needs a width of 2, `halves_b` a height (or
  north–south depth for a floor) of 2, `quadrants` both.
- If the size along a split axis is odd, the middle row goes to the panel on the low side: the west or north half,
  or the **bottom** half.
- **Distance**: with `halves_a` and `halves_b` each half moves by its own size along the split axis, or by
  `blocks-to-move` if you set it (both halves then move by the same amount). With `quadrants` each quarter always moves
  by its own size along both axes; `blocks-to-move` is ignored.

**Worked example**: a wall 4 blocks wide and 3 blocks tall.

- `halves_a`: each half is 2 wide and slides 2 blocks sideways. You need a free pocket 2 wide and 3 tall on each
  side of the doorway.
- `halves_b`: the bottom half gets the middle row (2 rows) and slides 2 blocks **down into the ground**; the top half
  (1 row) slides 1 block up. You need 2 free rows below and 1 above.
- `quadrants` on a 4 × 4 wall: each 2 × 2 quarter moves 2 blocks sideways and 2 blocks up or down at the same time.

## Creating it

1. Type `/dp create split_door [name]`. You receive the creation stick.
2. If you did not give a name, type it in chat.
3. Click the first corner of the door with the stick.
4. Click the opposite corner.
5. **Split mode**: chat says **Choose Split mode (current: ...):**. Click one of the modes (only the modes that fit
   the door are listed), or **[Continue]** to keep the current one.
6. Add triggers if you want, then click **[Create]** on the summary.

There is no pivot and no direction step. The general procedure is on [Creating a door](../creating.md).

## Type options

| | |
|---|---|
| Option | `mode` |
| Command | `/dp set <door> mode <mode>` with `halves_a`, `halves_b` or `quadrants` |
| Menu | **Type settings** → **Split mode** (left click next, right click previous; modes that do not fit are skipped) |
| Default | `halves_a` |
| When | Only while the door is closed and not moving |

The split door also uses the general **blocks to move** setting (`/dp set <door> blocks-to-move <number>`, `0` to
`limits.max-blocks-to-move`, `0` = each half moves by its own size). It has no effect with `quadrants`.

## On the move

- **Speed**: `distance / (speed.blocks-per-second × speed multiplier)` seconds, using the panel that moves farthest.
- **Checks**: only the end positions of the panels are checked, not the cells they pass through.
- **Entities**: see [Collisions](../movement.md#collisions).

## Sounds

Config section `sounds.split_door`:

| Sound | Default |
|---|---|
| `start` | `minecraft:block.piston.extend`, volume 0.8, pitch 1.0 |
| `loop` | none |
| `end` | `minecraft:block.piston.contract`, volume 0.8, pitch 1.0 |

See [Sounds](../movement.md#sounds) for the format.

## Tips and common mistakes

!!! warning "Mode refused"
    **The door is too small for this split mode: each split side needs at least 2 blocks.** A 1-wide door cannot use
    `halves_a` or `quadrants`; a 1-tall door cannot use `halves_b` or `quadrants`.

!!! warning "Stacked halves on a wall go into the ground"
    With `halves_b` on a standing wall the bottom half moves down. Dig a slot for it, and fill the slot with barrier
    blocks so nobody falls in while the door is closed. See [Passable blocks](../movement.md#passable-blocks).

!!! tip "Iris hatch"
    A square floor with `quadrants` opens like an iris: the four quarters slide outwards at the same level, into the
    four corner areas around the hatch. Keep those areas free; barrier blocks there are fine and keep the floor
    walkable while the hatch is closed.
