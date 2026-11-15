# Flag

The flag waves continuously while it is active: a wave runs from the pole to the free end, and the free end moves the
most. This page explains how to build it, where the pole goes and how to tune the wave. Creation id: `flag`.

## What it does

The flag is a perpetual type:

| State | What you see |
|---|---|
| Inactive (0) | The blocks where you built them |
| Active (1) | The blocks wave sideways, out of the plane of the flag, as long as a player is nearby |

- **Toggle** and **Open** start it; **Close** stops it; **Follow state** runs it while the source is active.
- When you stop it, the blocks go back to their built position at once.
- While active, it is only animated when its chunks are loaded and a player is within `animation.viewer-range`
  blocks (default 64). See [Perpetual types](../movement.md#perpetual-types).

How the wave works: a block at distance `d` blocks from the pole column moves sideways by
`amplitude × (d / L) × sin(k·d − ω·t)`, where `L` is the width of the flag minus 1,
`k = 2π / wavelength` and `ω = 2π / period-ticks`. Each block also turns slightly to follow the slope of the wave.
The pole column itself (`d = 0`) does not move.

## How to build it

- **Shape**: a vertical wall, 1 block thick, at least 2 blocks wide.
- **Pole**: one of the two vertical edges of the flag. That column stays still.
- **Front**: none.
- **Free space**: none is checked. The waving blocks pass through anything next to the flag.

**Worked example**: a banner of wool 6 blocks wide and 4 blocks tall, on a pole at its west end.

1. Build the pole (fences, for example) one column west of the wool, outside the flag, or use the first wool column
   as the edge that stays still.
2. Create the flag selecting the 6 × 4 × 1 wool, and click a block of the west column as pivot.
3. With the default settings the free (east) end swings up to 0.6 blocks to each side, the wave is 6 blocks long and
   a full wave cycle takes 40 ticks (2 seconds).

## Creating it

1. Type `/dp create flag [name]`. You receive the creation stick.
2. If you did not give a name, type it in chat.
3. Click the first corner of the flag with the stick.
4. Click the opposite corner.
5. **Pivot**: click any block of the column at the pole end (one of the two end columns). Chat confirms
   **Pivot set at x, y, z.**
6. Add triggers if you want, then click **[Create]** on the summary.
7. Start it, for example with `/dp open <door>`.

There is no direction step. The general procedure is on [Creating a door](../creating.md).

## Type options

The flag has no per-door settings. The wave is set for every flag on the server in `config.yml`:

| Config key | Default | Allowed | Meaning |
|---|---|---|---|
| `types.flag.amplitude` | `0.6` | 0 to 4 | How far (in blocks) the free end moves to each side |
| `types.flag.wavelength` | `6.0` | 0.5 to 128 | Length of one wave, in blocks |
| `types.flag.period-ticks` | `40` | 2 to 1200 | Time of one full wave cycle, in ticks (20 ticks = 1 second) |

After changing these values, run `/dp admin reload`: running flags take the new wave at once, without jumping.
**Blocks to move** and the speed multiplier do not apply.

## On the move

- **Checks when starting**: every block must be allowed in a door (see [Blocks](../movement.md#which-blocks-can-move))
  and protection plugins must allow the door.
- **Collisions**: none. Players walk through the waving blocks.
- **Animation limit**: a flag with more blocks than `animation.max-animated-blocks`, or that would pass
  `animation.max-global-animated-blocks`, stays still while active.

## Sounds

Config section `sounds.flag`:

| Sound | Default |
|---|---|
| `start` | none |
| `loop` | `minecraft:block.wool.step`, volume 0.3, pitch 0.8, every 40 ticks |
| `end` | none |

See [Sounds](../movement.md#sounds).

## Tips and common mistakes

!!! warning "Pivot refused"
    **The pivot must be on one of the two vertical edges of the door.** Click a block of the first or last column.

!!! warning "Flag too narrow or thick"
    A 1-wide selection gives **The door must be at least 2 blocks wide.**; a thick or flat one gives **The door must
    be a vertical wall, one block thick.**

!!! tip "Calmer flag"
    Lower `types.flag.amplitude` (for example `0.3`) and raise `types.flag.period-ticks` (for example `80`).
