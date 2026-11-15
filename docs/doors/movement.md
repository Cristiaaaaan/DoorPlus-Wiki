# Movement and animation

This page explains what happens every time a door moves: the checks Door+ makes first, how the blocks are animated,
how fast they go, what happens to players in the way, and what to do when a door refuses to move. It is for server
owners and staff who build doors or tune `config.yml`.

All keys below are in `config.yml`. Every section except `storage` is applied by `/dp admin reload`. The complete
list of keys is on [Configuration](../reference/configuration.md).

## What happens when a door moves

When a player, a trigger, a command, auto-close or another plugin asks a door to move:

1. Door+ checks that the door may move now: movements not paused, door not locked, the player allowed, the door not
   already moving, no cooldown. See [Checks before moving](#checks-before-moving).
2. It loads the chunks of the start and end positions if needed and keeps them loaded until the movement ends.
3. It computes where every block goes and checks the end position: allowed blocks, free space, no other door, inside
   the world height, allowed by protection plugins.
4. It writes the movement to its anti-crash journal in the database.
5. It removes the blocks from the world and shows them as moving entities (the animation), unless the movement is
   [instant](#instant-movement).
6. At the end it places the blocks in their new position, saves the new state of the door and deletes the journal
   entry.

The open position is never stored as blocks: Door+ remembers where the door was built (its *closed* position) and
computes the other positions from the door type. The blocks themselves are read from the world each time the door
moves. Exceptions: an open [rolling shutter](types/rolling-shutter.md) and an active
[perpetual door](#perpetual-types) keep a copy of their blocks in the database.

## Checks before moving

Door+ makes these checks in this order and stops at the first one that fails. The message the player sees is listed
in [Why a door does not move](#why-a-door-does-not-move).

| # | Check | Fails when |
|---|---|---|
| 1 | Door and world | The door was deleted, or its world is not loaded |
| 2 | Paused | An administrator ran `/dp admin pause`, or the server is shutting down |
| 3 | Locked | The door is locked (`/dp lock <door>`). This also stops auto-close |
| 4 | Access | The player is not allowed to use the door. Requests without a player follow their own rule: console, API and auto-close are allowed; redstone needs **Redstone** enabled on the door and `triggers.redstone.enabled`. See [Privacy and roles](../access/privacy-and-roles.md) |
| 5 | Already there | The door is already in the requested state (for example "open" on an open door) |
| 6 | Busy | The door is moving, or still being prepared |
| 7 | Cooldown | Less than `movement.toggle-cooldown-ticks` since the last movement of this door ended |
| 8 | Blocks | A block of the door is not allowed in doors (see [Which blocks can move](#which-blocks-can-move)) |
| 9 | World height | The end position goes above or below the world's height limits |
| 10 | Free space | A cell of the end position is not free, belongs to another door, or (rotating types) a cell the door sweeps through is not free. See [Free space](#free-space) |
| 11 | Protection | A protection plugin such as WorldGuard denies the end position. See [WorldGuard](../integrations/worldguard.md) |
| 12 | Other plugins | Another plugin cancelled the movement through the Door+ API |

The perpetual types (windmill, flag, clock) skip checks 9 and 10 when they start.

## Free space

Every cell of the end position that the door does not already occupy must be **free**. Door+ sorts blocks into three
classes:

| Class | Config key | Default | What the door does |
|---|---|---|---|
| Replaceable | `blocks.replaceable` | Air (always), plus `WATER`, `LAVA`, `SNOW`, `FERN`, `SHORT_GRASS`, `TALL_GRASS`, `SEAGRASS`, `TALL_SEAGRASS` | Counts as empty. The door overwrites it and it does **not** come back |
| Passable | `blocks.passable` | `BARRIER`, `LIGHT`, `STRUCTURE_VOID` | Counts as free. The door takes its place and puts it back when it leaves. See [Passable blocks](#passable-blocks) |
| Obstacle | everything else | | The door does not move |

- A material in both lists counts as passable.
- Blocks with a block entity (chests, signs...) cannot be passable; Door+ ignores them in `blocks.passable` and warns
  in the console.

Door+ also refuses to move a door into a cell that belongs to **another door**, whether that door is standing still
or moving there at the same moment.

**Swept volume**: the big door, drawbridge, folding door and revolving door turn, so their blocks pass through cells
outside both the start and end positions (a quarter circle). With `movement.check-swept-volume: true` (default) those
cells must be free too. The types that slide (portcullis, sliding door, split door, platform) are checked only at the
end position: the cells between start and end are not checked.

!!! note
    When you create a door whose end position is blocked, Door+ still creates it and warns: **Warning: the
    destination of the door is obstructed. It will not move until the space is free.**

## Passable blocks

Passable blocks let a door move into a space that must stay closed while the door is away. Typical case: a
portcullis that sinks into the ground. You dig the slot and fill it with barriers, so nobody falls in while the
portcullis is up.

- **When the door arrives**, every passable block in its end position is remembered and replaced: by a door block,
  or by air if that cell of the door is empty. The barriers stay in place during the whole animation and are only
  replaced at the end, so the hole is never open.
- **When the door leaves**, every remembered block is put back at the start of the movement.
- A passable block that is inside the region when you create the door becomes part of the door and moves with it.
- If a player breaks a door block in a cell with a remembered barrier, the barrier comes back the next time the door
  moves.
- When a door is deleted, its remembered blocks are forgotten; the blocks stay as they are in the world.

## Which blocks can move

A block can be part of a door only if all of these are true:

| Rule | Config key | Default |
|---|---|---|
| Not in the blacklist | `blocks.blacklist` | `BEDROCK`, `END_PORTAL`, `END_PORTAL_FRAME`, `NETHER_PORTAL`, `END_GATEWAY`, `COMMAND_BLOCK`, `CHAIN_COMMAND_BLOCK`, `REPEATING_COMMAND_BLOCK`, `STRUCTURE_BLOCK`, `JIGSAW`, `REINFORCED_DEEPSLATE` |
| In the whitelist, if the whitelist is not empty | `blocks.whitelist` | Empty: every block not blacklisted is allowed |
| No block entity | `blocks.allow-tile-entities` | `false`. Chests, barrels, furnaces, signs, banners, hoppers, spawners and every other block with a block entity cannot be part of a door, so their content is never lost. The key is reserved: only `false` is accepted |

- Replaceable blocks (air, water, grass...) inside the region are ignored: they are not part of the door.
- Door+ checks the rules when you create the door (**The block ... at x, y, z cannot be part of a door.**) and again
  every time it moves. A forbidden block placed into the door later (for example a chest) stops the door with
  **... cannot move: the block at x, y, z cannot be animated.**
- **Turning blocks**: when a door turns around a vertical axis, blocks with a direction (stairs, doors, logs...) are
  turned the Minecraft way. Around a horizontal axis, logs and pillars change their axis and blocks with a facing turn
  it if the new facing is possible for that block; other blocks keep their state.
- **Neighbour updates**: the blocks are placed without physics. With `movement.update-neighbors: true` (default)
  Door+ then sends one block update around the edges of the door, so redstone and attached blocks next to it react.

## Animation engines

`animation.engine` chooses how moving blocks are shown:

| | `block_display` (default) | `falling_block` |
|---|---|---|
| Entities per block | 1 (a block display) | 2 (a falling block riding an invisible armor stand) |
| Rotation | Blocks turn smoothly with the door | Blocks stay straight while they travel along the arc |
| Rolling shutter rows | Shrink as they roll into the edge | Disappear at the edge |
| Smoothness | The client interpolates between updates | Moved at every update |
| Availability | Always | Door+ tests it once after start-up; if the test fails it uses `block_display` and logs **animation.engine is falling_block but that engine is unavailable; using block_display** |

Use `block_display` unless you specifically want the falling-block look. It needs half the entities and is the only
engine that shows rotation and shrinking.

Options of `block_display`:

| Key | Default | Allowed | Meaning |
|---|---|---|---|
| `animation.display.view-range` | `1.0` | 0.01 to 64 | View range of the moving blocks (the display entity view range setting) |
| `animation.display.fixed-brightness` | `true` | true/false | Each moving block keeps the light of the cell it started from, so it does not flicker |

**Hidden blocks**: a block whose six neighbours are all solid, light-blocking blocks of the same door gets no entity:
nobody can see it. It is still moved and placed at the end.

## Instant movement

A movement skips the animation and the blocks jump to their end position in the same tick when:

| Condition | Config key | Default |
|---|---|---|
| No player within this distance of the door (start and end positions) | `animation.viewer-range` | 64 blocks |
| A chunk of the door had to be loaded for the movement | | |
| The door has more animated blocks than this | `animation.max-animated-blocks` | 1500 |
| The animation entities of all moving doors on the server would pass this total | `animation.max-global-animated-blocks` | 10000 |
| The movement was requested as instant (for example auto-close in unloaded chunks) | | |

Hidden blocks do not count towards these limits. With `falling_block` each block counts as 2 entities in the global
total. Instant movements still run every check: a door that cannot move animated cannot move instantly either.

## Speed

The time a movement takes depends on the type:

| Movement | Duration in seconds |
|---|---|
| Sliding by `n` blocks (portcullis, sliding door, split door, platform, rolling shutter) | `n / (speed.blocks-per-second × m)` |
| Turning 90° (big door, drawbridge, revolving door per quarter turn) | the longer of `90 / (speed.degrees-per-second × m)` and `(1.571 × r) / (speed.max-tip-speed × m)` |
| Garage door | `height / (speed.blocks-per-second × m)` |
| Folding door | like turning 90°, with `r` = width of the hinge panel |

`m` is the door's **speed multiplier** and `r` is the distance from the axis to the farthest corner of the door. The
second part of the turning formula slows down big doors so their far end never moves faster than
`speed.max-tip-speed` blocks per second.

| Key | Default | Allowed |
|---|---|---|
| `speed.blocks-per-second` | `2.0` | 0.05 to 100 |
| `speed.degrees-per-second` | `45.0` | 1 to 3600 |
| `speed.max-tip-speed` | `6.0` | 0.1 to 200 |
| `animation.min-ticks` | `10` (0.5 s) | 1 to 1200 |
| `animation.max-ticks` | `600` (30 s) | 1 to 12000, not below `min-ticks` |

Every duration is rounded to ticks and then kept between `animation.min-ticks` and `animation.max-ticks`.

**Examples with the defaults**: a portcullis rising 4 blocks takes 2 seconds; a big door 4 wide takes 2 seconds; a big
door 16 wide (radius 15) takes about 3.9 seconds.

**Per-door multiplier**: `/dp set <door> speed <multiplier>`, or the **Speed** button of the door menu (left +0.25,
right −0.25). The value must be between `limits.speed-multiplier.min` (default 0.25) and
`limits.speed-multiplier.max` (default 4.0); 2.0 means twice as fast. The windmill, flag and clock do not use the
multiplier: see their pages.

## Easing and smoothness

| Key | Default | Allowed | Meaning |
|---|---|---|---|
| `animation.easing` | `EASE_IN_OUT` | `LINEAR`, `EASE_IN_OUT` | `EASE_IN_OUT` starts slowly, speeds up and slows down before the end; `LINEAR` moves at a constant speed |
| `animation.update-interval-ticks` | `2` | 1 to 5 | How often (in ticks) the moving blocks are updated. Lower is smoother and costs more |

## Collisions

The collision mode decides what moving blocks do to players and mobs during the animation.

| Mode | Name in menus | During the animation |
|---|---|---|
| `none` | **None** | Entities pass through the moving blocks |
| `shulker` | **Solid** | The moving blocks are solid: an invisible shulker rides along with every outer block of the door |
| `push` | **Push** | Entities touched by a moving block are pushed away |

- **Server default**: `collision.default-mode` (default `none`). `collision.allowed-modes` lists the modes doors may
  use (default all three); if the default mode is not in the list, the first allowed mode is used.
- **Per door**: `/dp set <door> collision <mode>` with `default`, `none`, `shulker` or `push`, or the
  **Collisions** button of the door menu (cycles through **default** and the allowed modes). Only allowed modes are
  accepted.

**Solid (`shulker`)**:

- Only outer blocks (at least one face not touching another block of the door) get a shulker.
- The shulker hitboxes are straight cubes, so for turning doors the collision is approximate.
- `collision.shulker.max-per-door` (default 200, 0 to 10000): if the door has more outer blocks, that movement uses
  `push` instead.
- Door+ tests at start-up whether the server supports the riding technique; if it does not, `push` is used.
- The shulkers cannot be hurt, targeted or teleported.

**Push**:

| Key | Default | Allowed | Meaning |
|---|---|---|---|
| `collision.push.non-players` | `true` | true/false | Also push mobs and dropped items. Players (except spectators) are always pushed |
| `collision.push.strength` | `1.2` | 0 to 10 | Push speed = speed of the block × strength |
| `collision.push.max-velocity` | `1.5` | 0.05 to 10 | Highest push speed |

**Eviction at the end**: with `collision.evict-on-finish: true` (default), in every mode, a player or mob that ends up
inside a solid block of the door is teleported to the nearest free spot: first further along the direction the door
moved (up to 32 blocks), then upwards (up to 32 blocks), then around it (up to 3 blocks). A free spot is two passable
blocks on top of each other. Entities riding something are not moved.

**Platforms** carry what stands on them, whatever the mode; see [Platform / elevator](types/platform.md#on-the-move).

**Perpetual types** have no collisions and no eviction.

## Perpetual types

The [windmill](types/windmill.md), [flag](types/flag.md) and [clock](types/clock.md) keep moving while they are
**Active**.

- **Starting**: Door+ saves a copy of the door's blocks in the database. The state becomes **Active**.
- **Animated or resting**: an active door is animated only while its chunks are loaded and a player is within
  `animation.viewer-range` blocks. Otherwise it *rests*: its blocks are in the world in their built position. Door+
  checks every `perpetual.check-interval-ticks` (default 20, allowed 1 to 1200) and switches between the two. The
  position depends only on the time since it was started, so it looks continuous when you come back.
- **Animation limits**: a door that would pass `animation.max-animated-blocks` or
  `animation.max-global-animated-blocks` keeps resting.
- **Changes while resting**: if someone changes the blocks of a resting door, the saved copy is updated the next time
  it is animated.
- **Stopping**: the windmill finishes its current turn first; the flag and the clock stop at once. The blocks go back
  to their built position, the saved copy is deleted and the state becomes **Inactive**.
- **Restart**: an active door stays active. At start-up, any empty cell of its area is refilled from the saved copy,
  then it is animated again when a player comes near.
- Follow state, auto-close and cooldowns work as for the other types.

## Auto-close

A door can close by itself some seconds after it opens.

| | |
|---|---|
| Command | `/dp set <door> auto-close <seconds>` (0 = off) |
| Menu | **Auto-close** in the door menu (left +1 s, right −1 s, shift ×10) |
| Maximum | `limits.max-auto-close-seconds` (default 600) |
| Default for new doors | `defaults.auto-close-seconds` (default 0) |

- The countdown starts every time the door arrives in its open state, whatever opened it. It works for open/closed
  types and perpetual types (it stops them); it has no effect on platforms and revolving doors.
- It does not start while a trigger with the **Follow state** action is active (a player still in the proximity zone,
  a lever still on); it starts when the last one turns off.
- The countdown is cancelled when the door starts moving.
- Auto-close does not need access, but it does not close a **locked** door.
- If the door's chunks are not loaded when the countdown ends: with `movement.auto-close-unloaded: true` (default) the
  door closes instantly; with `false` it closes when one of its chunks loads again.
- If a cooldown is running, auto-close tries again when it ends.
- After a restart, doors that were left open start a new full countdown.

## Cooldowns and locked doors

- `movement.toggle-cooldown-ticks` (default 0 = off, up to 72000) is a pause after every movement of a door, counted
  from the end of the movement. Requests during the pause are ignored without a message. Triggers have their own
  cooldown; see [Triggers](../access/triggers.md).
- A **locked** door cannot be moved by anyone, by any cause, including auto-close and redstone. Players see **... is
  locked.** in the action bar. Unlock it with `/dp unlock <door>` or the **Lock** button of the door menu.

## Restart, shutdown and crashes

- **Normal shutdown or reload of the plugin**: running animations finish instantly (the blocks are placed at the end
  position and the new state is saved). Active perpetual doors put their blocks back in the world and stay active.
  Requests made while the server is stopping are refused.
- **Crash**: before removing any block Door+ writes the movement to a journal in the database. At the next start it
  completes every interrupted movement: the blocks are placed at the end position and the door gets its new state.
  The console logs **Journal: recovered X of Y interrupted door movements**. A journal entry is kept for later if the
  door's world is not loaded, or if a block in it is no longer valid (the console says why, with the door id). A
  door being recovered counts as busy.
- **Left-over entities**: moving-block entities are never saved with the world. If some remain anyway, Door+ removes
  them at start-up and whenever their chunk loads; `/dp admin cleanup` removes them in all loaded chunks.

## Admin controls

| Command | Permission (default: operators) | Effect | Message |
|---|---|---|---|
| `/dp admin pause` | `doorplus.admin.pause` | Pauses all door movements; run it again to resume. New requests are refused, running animations finish normally | **Door movements paused.** / **Door movements resumed.** |
| `/dp admin stopall` | `doorplus.admin.stopall` | Finishes every running animation instantly. Animated perpetual doors go to rest; they stay active and start again at the next check if a player is near | **Finished N running animations.** |
| `/dp admin cleanup` | `doorplus.admin.cleanup` | Removes left-over moving-block entities in loaded chunks | **Removed N orphan entities.** |
| `/dp admin reload` | `doorplus.admin.reload` | Reloads `config.yml` (except `storage`) and the language files | **Configuration and language files reloaded.** |

## Sounds

Each type has its own sounds in `sounds.<type>`, for example `sounds.big_door`. The defaults are listed on each type
page.

```yaml
sounds:
  enabled: true                 # false = no door sounds at all
  big_door:
    start: { sound: 'minecraft:block.wooden_door.open', volume: 1.0, pitch: 0.7 }
    loop:  { sound: '', volume: 0.0, pitch: 1.0, interval-ticks: 10 }
    end:   { sound: 'minecraft:block.wooden_door.close', volume: 1.0, pitch: 0.7 }
```

| Field | Meaning |
|---|---|
| `sound` | Namespaced sound key. Empty = no sound |
| `volume` | 0 or more |
| `pitch` | Kept between 0.5 and 2.0 |
| `interval-ticks` | `loop` only: ticks between two plays (at least 1, default 10) |

- `start` plays when the movement starts, at the centre of the start position; `loop` plays during the animation and
  `end` at the end, at the centre of the end position. Sounds play in the **Blocks** sound category.
- Perpetual types: `start` when switched on, `loop` while animated, `end` when stopped.

## Why a door does not move

These are the messages the player who used the door (or the command sender) can receive, and how to fix each one.

| Message | Cause | Fix |
|---|---|---|
| **Door movements are paused by an administrator.** | `/dp admin pause` is on | Run `/dp admin pause` again |
| **... is locked.** (action bar) | The door is locked | `/dp unlock <door>` |
| **You cannot use ....** (action bar) | The player has no access | Change the privacy or add the player; see [Privacy and roles](../access/privacy-and-roles.md) |
| **... cannot move: the block at x, y, z cannot be animated.** | A blacklisted block, a block outside the whitelist, or a block with a block entity is in the door | Remove that block, or change `blocks.blacklist` / `blocks.whitelist` |
| **... cannot move: something is in the way at x, y, z.** | That cell of the end position (or of the swept area) is not free, or belongs to another door | Remove the block, move the other door, or change the direction |
| **... cannot move beyond the height limits of the world.** | The end position is above or below the world limits | Change the direction, the blocks to move or the stops |
| **... cannot move here: denied by ....** | A protection plugin (named in the message) denies the end position | Allow the door in that plugin; see [WorldGuard](../integrations/worldguard.md) |
| **... could not move because of an error. Details are in the server console.** | Unexpected error | Check the console and report it |
| **The world of ... is not loaded.** (commands only) | The door's world is not loaded | Load the world |
| **Door not found.** (commands only) | The door does not exist | Check the name or id |

No message is sent when:

- the door is already in the requested state (for example opening an open door);
- the door is already moving;
- the door's cooldown (`movement.toggle-cooldown-ticks`) is running;
- another plugin cancelled the movement.

If a door seems to do nothing, check these four cases, then `/dp admin debug <door>`, which writes the internal state
of the door to the console.
