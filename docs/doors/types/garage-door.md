# Garage door

The garage door is an up-and-over door: it slides up out of the doorway and then runs back horizontally, ending flat
just above the doorway, inside the building. This page explains how to build it and where the space for the open door
must be. Creation id: `garage_door`.

## What it does

| State | Position |
|---|---|
| Closed (0) | The blocks where you built them: a vertical wall |
| Open (1) | A horizontal layer one block above the top of the door, as wide as the door and as deep as the door was tall |

While opening, every row follows an L-shaped track: straight up to the height one above the door, then horizontally
in the opening direction. Each row turns 90° as it goes round the corner, so slabs, stairs and trapdoors end up lying
flat. The bottom row ends right above the doorway, the top row ends farthest inside.

## How to build it

- **Shape**: a vertical wall, 1 block thick.
- **Front**: none, but the direction is always one of the two sides of the wall: the side the door folds back into
  (normally the inside of the garage).
- **Free space**: the open layer must be free. It sits one block above the top row of the door, starts **in the line
  of the wall**, directly above the doorway, and goes as many blocks deep as the door is tall.

**Worked example**: a garage door 3 blocks wide and 3 blocks tall, in a wall running east–west, with the garage to
the north. Direction **North**.

- Open, the door is a flat 3 × 3 layer at the height of the row just above the door: the first row of the layer is
  in the wall line, the other two go 2 blocks into the garage.
- So the wall above the doorway must leave that row free: put the lintel of the wall one block higher, at the
  second row above the door.

## Creating it

1. Type `/dp create garage_door [name]`. You receive the creation stick.
2. If you did not give a name, type it in chat.
3. Click the first corner of the door with the stick.
4. Click the opposite corner.
5. **Direction**: click one of the two sides of the wall (**North**/**South** for a wall running east–west,
   **East**/**West** for a wall running north–south).
6. Add triggers if you want, then click **[Create]** on the summary.

There is no pivot step. The general procedure is on [Creating a door](../creating.md).

## Opening direction

| Wall runs | Valid directions | Default order |
|---|---|---|
| East–west | North, South | North, then South |
| North–south | East, West | East, then West |

**Default**: the first direction whose open layer is free; if neither is free, the first one. Change it later with
`/dp set <door> direction <direction>` or the **Direction** button of the door menu, while the door is closed and not
moving.

## Type options

The garage door has no type settings. **Blocks to move** does not apply to this type. General settings are on
[Editing and deleting](../editing.md).

## On the move

- **Speed**: `height / (speed.blocks-per-second × speed multiplier)` seconds. The 3-tall door above takes 1.5 seconds
  with the default 2 blocks per second.
- **Checks**: the open layer must be free. The track runs through the door's own cells and the open layer, so nothing
  else is checked.
- **Entities**: see [Collisions](../movement.md#collisions).

## Sounds

Config section `sounds.garage_door`:

| Sound | Default |
|---|---|
| `start` | `minecraft:block.iron_trapdoor.open`, volume 1.0, pitch 0.6 |
| `loop` | `minecraft:block.chain.place`, volume 0.5, pitch 0.9, every 8 ticks |
| `end` | `minecraft:block.iron_trapdoor.close`, volume 1.0, pitch 0.6 |

See [Sounds](../movement.md#sounds) for the format.

## Tips and common mistakes

!!! warning "Lintel right above the door"
    The most common mistake: the wall block directly above the doorway is in the open layer. The message
    **... cannot move: something is in the way at x, y, z.** points at it. Leave one free row above the door, inside
    the wall line too.

!!! warning "Thick door"
    The garage door must be exactly 1 block thick. A thicker selection is refused with **The door must be a vertical
    wall, one block thick.**

!!! tip "Low ceiling"
    The garage needs a free layer as deep as the door is tall. If there is no such room, use a
    [rolling shutter](rolling-shutter.md): it stores the rolled-up rows and needs no space.
