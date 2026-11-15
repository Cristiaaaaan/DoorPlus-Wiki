# Big door

The big door swings 90° around a vertical hinge, like an ordinary door or a castle gate. This page explains how to
build one, what to click during creation and how it behaves. Creation id: `big_door`.

## What it does

| State | Position |
|---|---|
| Closed (0) | The blocks where you built them |
| Open (1) | The blocks turned 90° around the hinge column, clockwise or counter-clockwise seen from above |

Opening turns the door one way; closing turns it back. Stairs, logs, trapdoors and other blocks with a direction turn
with the door, so they keep facing the right way.

## How to build it

- **Shape**: any box. Usually a wall 1 block thick, but a thick door works too.
- **Front**: the big door has no front; it can open to either side. You choose the side with the direction.
- **Hinge**: any column of the door. Usually you use one of the two end columns. A column in the middle makes the door
  pivot around its centre: one part swings out on each side.
- **Free space**: the open position must be free, and so must every block the door passes through while it swings
  (a quarter circle around the hinge). See [Free space](../movement.md#free-space).

**Worked example**: a gate 4 blocks wide and 5 blocks tall, 1 block thick, running east–west, with the hinge on the
west end.

- Closed, the gate fills 4 × 5 blocks along the wall.
- Open clockwise, it stands 4 blocks deep towards the south, 5 blocks tall, with the hinge column unchanged. Open
  counter-clockwise, it goes towards the north.
- While it swings, the far end draws a quarter circle with a radius of 3 blocks, so the quarter circle between the
  wall and the open position must be empty.

## Creating it

1. Type `/dp create big_door [name]`. You receive the creation stick.
2. If you did not give a name, type it in chat.
3. Click the first corner of the door with the stick (left or right click).
4. Click the opposite corner. The chat shows the size of the region.
5. **Pivot**: click any block of the hinge column. Only its column matters; the height you click at does not.
   Chat confirms **Pivot set at x, y, z.**
6. **Direction**: click **Clockwise** or **Counter-clockwise**. The pre-selected one is the default (see below).
7. Add triggers if you want, then click **[Create]** on the summary.

The general procedure, the preview and the trigger step are on [Creating a door](../creating.md).

## Opening direction

| Direction | Meaning (seen from above) |
|---|---|
| **Clockwise** (`cw`) | North → east → south → west |
| **Counter-clockwise** (`ccw`) | North → west → south → east |

For a wall running east–west:

| Hinge on the... | Clockwise opens towards | Counter-clockwise opens towards |
|---|---|---|
| West end | South | North |
| East end | North | South |

**Default**: clockwise if its open position is free, otherwise counter-clockwise if that one is free, otherwise
clockwise. You can change the direction later with `/dp set <door> direction cw|ccw` or the **Direction** button of
the door menu, but only while the door is closed and not moving.

## Type options

The big door has no type settings. The general settings (speed, auto-close, collisions...) are on
[Editing and deleting](../editing.md). **Blocks to move** does not apply to this type.

## On the move

- **Speed**: a quarter turn takes the longer of `90 / (speed.degrees-per-second × speed multiplier)` seconds and
  `(1.571 × radius) / (speed.max-tip-speed × speed multiplier)` seconds, where the radius is the distance from the
  hinge to the farthest corner of the door. With the defaults (45 °/s, 6 blocks/s) a door up to 8 blocks wide takes 2 seconds; a
  16-wide door (radius 15) takes about 3.9 seconds, so its far end does not move faster than 6 blocks per second.
- **Checks**: the open position and the swept quarter circle must be free (the swept check can be turned off with
  `movement.check-swept-volume: false`).
- **Entities**: what happens to players in the way depends on the collision mode; see
  [Collisions](../movement.md#collisions).

## Sounds

Config section `sounds.big_door`:

| Sound | Default |
|---|---|
| `start` | `minecraft:block.wooden_door.open`, volume 1.0, pitch 0.7 |
| `loop` | none |
| `end` | `minecraft:block.wooden_door.close`, volume 1.0, pitch 0.7 |

See [Sounds](../movement.md#sounds) for the format.

## Tips and common mistakes

!!! warning "The door does not open and says something is in the way"
    The message **... cannot move: something is in the way at x, y, z.** gives the first blocking block. It is often a
    block in the quarter circle the door sweeps, not in the open position itself: a lantern, a torch or a wall
    corner next to the hinge.

!!! tip "Wrong side"
    If the door opens to the wrong side, close it and change the direction with `/dp set <door> direction ccw` (or
    `cw`).

!!! tip "Double gate"
    For two wings that open together, create two big doors with the hinge on opposite ends and opposite directions,
    then put both in a [door group](../../access/groups.md) and link your trigger to the group.
