# Folding door

The folding door folds in two like a book while it swings open, so it needs only about half the room of a big door of
the same width. This page explains how to build it, where it ends up and what to click. Creation id: `folding_door`.

## What it does

The door is split into two panels:

- **Panel 1**: the half next to the hinge (the larger half if the width is odd).
- **Panel 2**: the other half.

| State | Position |
|---|---|
| Closed (0) | The blocks where you built them: a flat wall |
| Open (1) | Panel 1 stands at 90° to the wall, starting at the hinge. Panel 2 is folded back flat against it, one block further into the doorway |

While it opens, panel 1 swings like a [big door](big-door.md) and panel 2 turns the opposite way around the joint, so
the two panels close up like the pages of a book. Blocks with a direction turn with their panel.

## How to build it

- **Shape**: a vertical wall, 1 block thick, at least 2 blocks wide.
- **Front**: none, but the direction is one of the two sides of the wall: the side the door folds towards.
- **Hinge**: one of the two end columns of the door.
- **Free space**: the open position must be free, and so must the cells the panels sweep while folding.

**Worked example**: a door 4 blocks wide and 3 blocks tall in a wall running east–west, hinge on the west end,
direction **South**.

- Panel 1 is the 2 west columns, panel 2 the 2 east columns.
- Open, the door is a 2 × 2 block (3 tall) on the south side of the doorway: panel 1 in the hinge column, going 2
  blocks south; panel 2 right next to it, one column to the east. The other 2 columns of the doorway are free.
- With a width of 5, panel 1 is 3 columns and goes 3 blocks south; panel 2 (2 columns) lies against the far
  2 blocks of panel 1.

## Creating it

1. Type `/dp create folding_door [name]`. You receive the creation stick.
2. If you did not give a name, type it in chat.
3. Click the first corner of the door with the stick.
4. Click the opposite corner.
5. **Pivot**: click any block of one of the two end columns. That column becomes the hinge. Chat confirms
   **Pivot set at x, y, z.**
6. **Direction**: click one of the two sides of the wall (**North**/**South** for a wall running east–west,
   **East**/**West** for a wall running north–south).
7. Add triggers if you want, then click **[Create]** on the summary.

The general procedure is on [Creating a door](../creating.md).

## Opening direction

| Wall runs | Valid directions | Default order |
|---|---|---|
| East–west | North, South | North, then South |
| North–south | East, West | East, then West |

**Default**: the first direction whose open position is free; if neither is free, the first one. Change it later with
`/dp set <door> direction <direction>` or the **Direction** button of the door menu, while the door is closed and not
moving.

## Type options

The folding door has no type settings. **Blocks to move** does not apply to this type. General settings are on
[Editing and deleting](../editing.md).

## On the move

- **Speed**: like a quarter turn of a big door whose radius is the width of panel 1: the longer of
  `90 / (speed.degrees-per-second × multiplier)` and `(1.571 × panel 1 width) / (speed.max-tip-speed × multiplier)`
  seconds. With the defaults, doors up to 14 blocks wide take 2 seconds.
- **Checks**: the open position and the swept cells must be free (the swept check follows
  `movement.check-swept-volume`).
- **Entities**: see [Collisions](../movement.md#collisions).

## Sounds

Config section `sounds.folding_door`:

| Sound | Default |
|---|---|
| `start` | `minecraft:block.wooden_trapdoor.open`, volume 1.0, pitch 0.8 |
| `loop` | none |
| `end` | `minecraft:block.wooden_trapdoor.close`, volume 1.0, pitch 0.8 |

See [Sounds](../movement.md#sounds) for the format.

## Tips and common mistakes

!!! warning "Pivot refused"
    **The pivot must be on one of the two vertical edges of the door.** Click a block of the first or last column,
    not one in the middle.

!!! warning "Door too narrow or too thick"
    A 1-wide selection gives **The door must be at least 2 blocks wide.**; a thick one gives **The door must be a
    vertical wall, one block thick.**

!!! tip "Bi-fold pair"
    For a wide opening, create two folding doors with the hinges on the two outer ends and the same direction, and put
    them in a [door group](../../access/groups.md).
