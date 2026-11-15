# Clock

The clock is a perpetual type with two hands that show the time of the Minecraft world or the real time. This page
explains how to build the hands, how Door+ decides which side is the front and how to pick the time source. Creation
id: `clock`.

## What it does

| State | What you see |
|---|---|
| Inactive (0) | The hands where you built them, both pointing up (12 o'clock) |
| Active (1) | The hands show the current time, as long as a player is nearby |

- **Toggle** and **Open** start it; **Close** stops it; **Follow state** runs it while the source is active.
- When you stop it, the hands go back to 12 o'clock at once.
- The minute hand makes a full turn every hour, the hour hand every 12 hours. Both move smoothly and turn clockwise as
  seen from the front.
- While active, it is only animated when its chunks are loaded and a player is within `animation.viewer-range`
  blocks (default 64). See [Perpetual types](../movement.md#perpetual-types).

## How to build it

The clock is a box 2 blocks thick with a square face:

- **Thickness**: exactly 2 blocks, east–west or north–south.
- **Face**: vertical and square: width = height (for example 7 × 7).
- **Front layer** (the layer on the front side): the **minute hand**.
- **Back layer**: the **hour hand**.
- **Axis**: through the centre of the face. Use an odd size (5, 7, 9...) so it passes through the centre block.
- **Rest position**: build both hands pointing straight **up** from the centre (12 o'clock).
- **What turns**: every block in the front layer turns as the minute hand, every block in the back layer as the hour
  hand. Build the dial, the numbers and the frame **outside** the selection, for example behind it.
- **Free space**: none is checked. The hands pass through anything next to them.

### Which side is the front

The front is the side **you are standing on when you click the second corner**: Door+ makes the clock face you. Stand
in front of the clock, looking at it, when you click.

If you click while looking along the face instead (so the side you face does not match the thickness of the clock),
Door+ uses the west side as the front for a clock 2 blocks thick east–west, or the north side for one 2 blocks thick
north–south.

**Worked example**: a clock 7 × 7, 2 blocks thick north–south, on the south wall of a tower, read from the south.

1. On the tower wall, build the dial (7 × 7 or bigger) one block further north than the clock, so it is outside the
   selection.
2. Back layer (next to the dial): the hour hand, the centre block and 2 blocks above it.
3. Front layer (south): the minute hand, the centre block and 3 blocks above it.
4. Create the clock: select the 7 × 7 × 2 box and, standing south of it and looking north, click the second corner.

## Creating it

1. Type `/dp create clock [name]`. You receive the creation stick.
2. If you did not give a name, type it in chat.
3. Click the first corner of the clock with the stick.
4. Stand in front of the clock, looking at it, and click the opposite corner.
5. Add triggers if you want, then click **[Create]** on the summary.
6. Start it, for example with `/dp open <door>`.

There is no pivot, direction or parameter step. The general procedure is on [Creating a door](../creating.md).

To change the front later, redefine the region (**Redefine region** in the door menu, or `/dp redefine <door>`) and
click the second corner from the other side.

## Type options

| | |
|---|---|
| Option | `time-source` |
| Command | `/dp set <door> time-source world` or `real` |
| Menu | **Type settings** → **Time source** (click to switch; the button shows **World time** or **Real time**) |
| Values | `world`: the time of the Minecraft world the clock is in (world tick 0 = 06:00). `real`: the real time |
| Default | `world` |
| When | Any time. A running clock switches source at once: the hands move to the new time |

Config key `types.clock.timezone` (default `system`) sets the time zone for real time: `system` uses the server's
zone; otherwise use a zone id such as `Europe/Rome` or `UTC`. An invalid zone is reported in the console and the
server's zone is used. It is shared by all clocks.

**Blocks to move** and the speed multiplier do not apply.

## On the move

- **Checks when starting**: every block must be allowed in a door (see [Blocks](../movement.md#which-blocks-can-move))
  and protection plugins must allow the door.
- **Collisions**: none.
- **Animation limit**: a clock with more blocks than `animation.max-animated-blocks`, or that would pass
  `animation.max-global-animated-blocks`, stays still while active.

## Sounds

Config section `sounds.clock`:

| Sound | Default |
|---|---|
| `start` | `minecraft:block.note_block.hat`, volume 0.6, pitch 1.0 |
| `loop` | `minecraft:block.note_block.hat`, volume 0.4, pitch 1.8, every 20 ticks (a tick-tock every second) |
| `end` | `minecraft:block.note_block.hat`, volume 0.6, pitch 1.0 |

See [Sounds](../movement.md#sounds).

## Tips and common mistakes

!!! warning "Wrong shape"
    **A clock must be 2 blocks thick with a square vertical face.** Check that the selection is exactly 2 deep and as
    wide as it is tall.

!!! warning "The clock runs backwards"
    The front is on the wrong side: you clicked the second corner from behind. Redefine the region and click from the
    side people read the clock from.

!!! warning "The whole face spins"
    Blocks of the dial were inside the selection. Only the hands may be in the two layers.
