# Windmill

The windmill spins continuously around an axis through its centre while it is active: windmill sails, a water wheel,
a fan, or a carousel when it is built flat. This page explains how to build it, how to choose the spinning direction
and how to set its speed. Creation id: `windmill`.

## What it does

The windmill is a perpetual type:

| State | What you see |
|---|---|
| Inactive (0) | The blocks where you built them |
| Active (1) | The blocks spin around the centre at a steady speed, as long as a player is nearby |

- **Toggle** and **Open** start it; **Close** stops it; **Follow state** runs it while the source is active.
- When you stop it, it keeps turning until it has completed its current turn, then the blocks are placed back in
  their built position. At the default 30 degrees per second a full turn takes 12 seconds, so a stop can take up to
  12 seconds.
- While active, it is only animated when its chunks are loaded and a player is within `animation.viewer-range`
  blocks (default 64). Otherwise its blocks rest in the world in their built position. See
  [Perpetual types](../movement.md#perpetual-types).

## How to build it

- **Shape**: flat, 1 block thick, with a **square face**: the two sides that are not 1 block must be equal (for
  example 7 × 7 × 1).
- **Axis**: perpendicular to the face, through its centre. With an odd size the axis passes through the centre block;
  with an even size it passes between four blocks.

    | Built as | Spins like |
    |---|---|
    | A wall 1 block thick north–south | Sails facing north and south |
    | A wall 1 block thick east–west | Sails facing east and west |
    | A floor, 1 block thick vertically | A carousel, around a vertical axis |

- **What turns**: every block inside the selection, including the hub. Put the tower or the axle outside the
  selection.
- **Free space**: none is checked. The spinning blocks pass through anything around them, so keep the circle around
  the windmill clear for a clean look.

**Worked example**: sails 7 × 7 blocks, 1 block thick north–south, mounted on the north face of a tower.

1. Build the sails in a 7 × 7 square with the hub block in the centre. Leave the corners of the square empty (or not,
   they turn too).
2. Create the windmill selecting only the 7 × 7 × 1 square, not the tower.
3. Choose **Clockwise**: the sails turn clockwise for a player standing on the south side, looking north at them.

## Creating it

1. Type `/dp create windmill [name]`. You receive the creation stick.
2. If you did not give a name, type it in chat.
3. Click the first corner of the windmill with the stick.
4. Click the opposite corner.
5. **Direction**: click **Clockwise** or **Counter-clockwise**.
6. Add triggers if you want, then click **[Create]** on the summary.
7. Start it, for example with `/dp open <door>`.

The speed is not asked during creation. The general procedure is on [Creating a door](../creating.md).

## Spinning direction

Clockwise and counter-clockwise are seen from a fixed side, whatever side the windmill faces:

| Windmill 1 block thick... | Clockwise as seen from |
|---|---|
| North–south | The south side, looking north |
| East–west | The east side, looking west |
| Vertically (flat) | Above, looking down |

**Default**: Clockwise. Change it later with `/dp set <door> direction cw|ccw` or the **Direction** button of the
door menu, while the windmill is inactive and not moving.

## Type options

| | |
|---|---|
| Option | `degrees-per-second` |
| Command | `/dp set <door> degrees-per-second <number>` (decimals allowed) |
| Menu | **Type settings** → **Rotation speed** (left +1, right −1, shift ×10) |
| Values | `1` to `360` |
| Default | `types.windmill.degrees-per-second` (default 30), limited to 1–360 |
| When | Any time. A running windmill changes speed at once, without jumping |

The door's speed multiplier and the `speed.*` settings do not affect windmills. **Blocks to move** does not apply.

Config key `types.windmill.degrees-per-second` sets the speed of windmills that have no own value. After changing it,
`/dp admin reload` applies it to running windmills too.

## On the move

- **Checks when starting**: every block must be allowed in a door (see [Blocks](../movement.md#which-blocks-can-move))
  and protection plugins must allow the door; nothing else.
- **Collisions**: none. Players walk through the spinning blocks.
- **Animation limit**: if the windmill has more blocks than `animation.max-animated-blocks`, or the server total would
  pass `animation.max-global-animated-blocks`, it stays still (in its built position) even while active.

## Sounds

Config section `sounds.windmill`:

| Sound | Default |
|---|---|
| `start` | `minecraft:block.wood.place`, volume 0.8, pitch 0.6 |
| `loop` | `minecraft:block.bamboo_wood.step`, volume 0.4, pitch 0.6, every 30 ticks |
| `end` | `minecraft:block.wood.place`, volume 0.8, pitch 0.6 |

`start` plays when the windmill is switched on, `end` when it stops, `loop` while it is animated. See
[Sounds](../movement.md#sounds).

## Tips and common mistakes

!!! warning "Face not square"
    **The two sides of the face must be equal.** A 7 × 6 selection is refused.

!!! warning "Stopping takes a while"
    A slow windmill finishes its turn before stopping: at 1 degree per second that is up to 6 minutes. During that
    time the door counts as moving.

!!! tip "Odd sizes"
    Use an odd size (5, 7, 9...) so the axis passes through the centre block and the sails turn around the hub.
