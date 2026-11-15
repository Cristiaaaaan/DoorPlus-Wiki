# Creating a door

This page describes the guided creation of a door in full: how to start it, every step for every door type,
the preview, how it ends, and every error you can get with its fix. It is for players who create doors and
for staff who help them.

For a short hands-on example, see [Your first door](../getting-started/first-door.md).

## Before you start

You can start a creation only if all of these are true. The checks run in this order; the first one that
fails stops the start with the message shown.

| Check | Message if it fails |
|---|---|
| The type is enabled in `types.<type>.enabled` | **Doors of type ... are disabled on this server.** |
| You have `doorplus.create.<type>` | **You do not have permission to do this.** |
| You own fewer doors than your limit (`limits.doors-per-player`, default 10) | **You already own the maximum number of doors (...).** |
| You are not already creating or redefining a door | **You are already creating a door. Finish it or type /door cancel.** |
| You have a free inventory slot | **You need a free inventory slot for the creation stick.** |

Your door limit can be raised per player with permissions, and `doorplus.admin.bypass.limits` ignores it.
See [Limits](../reference/limits.md).

## Start the creation

=== "Command"

    Type `/dp create <type> [name]`.

    - `<type>` is the type id: `big_door`, `drawbridge`, `portcullis`, `sliding_door`, `garage_door`,
      `split_door`, `folding_door`, `rolling_shutter`, `platform`, `revolving_door`, `windmill`, `flag` or
      `clock`. Case does not matter and `-` works instead of `_`. Tab completion lists only the enabled types
      you may create.
    - `[name]` is optional. If you leave it out, the first step asks for the name in chat.

    If the name you type is not valid, the command stops with
    **Invalid name: use 1-32 letters, digits, _ or -.** If you already own a door with that name, the
    creation starts and the first step asks for another name.

=== "Menu"

    1. Type `/dp` to open the main menu **Door+ — Your doors**.
    2. Click **Create a door**. The menu **Choose the door type** lists the enabled types you may create.
       If there are none, the button says **You cannot create any door type.**
    3. Click a type. The menu closes and the creation starts. The first step asks for the name in chat.

When the creation starts you receive the **Door+ creation stick** and the message
**Creating a ... Use the stick you just received.**

## The creation stick

- The stick goes in the slot in your hand if it is empty, otherwise in the first empty slot.
- Left click and right click on a block both select it. The click never breaks or uses the block.
- Clicking with the stick when no block is expected shows **Use the buttons or the chat to continue.** above
  the hotbar.
- You cannot drop the stick, move it in your inventory, put it in a container, craft with it or lose it on
  death.
- When the creation ends for any reason, the stick is removed. A creation stick found outside a creation is
  deleted.

## How the steps work

- Each instruction starts with **[step/total]**, for example **[3/7]**. The total can change after you
  select the region, because some steps depend on the shape (see the table below).
- Most steps show clickable chat buttons. **[Back]** returns to the previous step (it is missing on the first
  step). **[Cancel]** ends the creation.
- Only the newest buttons work. Clicking an older button shows **That button is no longer valid.**
- When a step asks for text in chat, your message is hidden from other players. Type `cancel` to abort the
  chat input. The chat input times out after 30 seconds.

## Steps in order

| # | Step | What you do | Shown for |
|---|---|---|---|
| 1 | Name | Type the name in chat | Only if you did not give a valid, free name in the command |
| 2 | First corner | Click a block with the stick | Every type |
| 3 | Second corner | Click the opposite corner | Every type |
| 4 | Pivot | Click a block of the region | `big_door`, `drawbridge`, `folding_door`, `flag` |
| 5 | Pivot axis | Click a second block on the pivot row | `drawbridge`, only when the axis is ambiguous |
| 6 | Direction | Click one of the offered directions | Types with more than one valid direction for this region |
| 7 | Parameters | Click a value or type one | `split_door` (split mode), `rolling_shutter` (visible rows) |
| 8 | Triggers | Collect triggers, or click **[Done]** | Every type |
| 9 | Confirm | Check the summary and click **[Create]** | Every type |

Steps per type:

| Type | Steps after the two corners |
|---|---|
| `big_door` | Pivot, Direction (Clockwise / Counter-clockwise) |
| `drawbridge` | Pivot, Pivot axis (only if ambiguous), Direction (two, depending on the axis) |
| `portcullis` | Direction (Up / Down) |
| `sliding_door` | Direction (North / South / East / West) |
| `garage_door` | Direction (two, perpendicular to the door), only if the region is a vertical wall one block thick |
| `split_door` | Parameters: split mode |
| `folding_door` | Pivot (on a vertical edge), Direction (two, perpendicular to the door) |
| `rolling_shutter` | Direction (in the plane of the shutter), Parameters: visible rows |
| `platform` | Direction (Up / Down / North / East / South / West) |
| `revolving_door` | Direction (Clockwise / Counter-clockwise) |
| `windmill` | Direction (Clockwise / Counter-clockwise) |
| `flag` | Pivot (on a vertical edge) |
| `clock` | None |

Then every type has Triggers and Confirm. The shape each type needs is explained on its page under
[Door types](types/index.md).

### 1. Name

Chat shows **Type the name of the door in chat (1-32 letters, digits, _ or -).**

- A name is 1 to 32 characters: letters, digits, `_` and `-`. No spaces.
- You cannot own two doors with the same name. Names are compared without case: `Gate` and `gate` are the
  same name. Other players can use the same name for their own doors.
- An invalid or taken name shows the error and asks again.
- Chat shows **Name: ...** when the name is accepted.

!!! warning
    Typing `cancel`, or waiting 30 seconds, during the name step ends the whole creation if no name was set
    yet (**Door creation cancelled.**).

### 2–3. Corners

1. **Click the first corner of the door with the stick.** Chat shows **First corner set at x, y, z.**
2. **Click the opposite corner of the door.** Chat shows
   **Second corner set at x, y, z. Region: AxBxC.** The size is east-west × height × north-south.

The two blocks are opposite corners of a box; every block inside the box belongs to the door. Air and the
blocks in `blocks.replaceable` (water, lava, snow layers, grass, ferns, seagrass) are not part of the door.

- The first corner cannot be in a world listed in `worlds.disabled`
  (**Doors cannot be created in the world ...**).
- The second corner must be in the same world as the first
  (**Both corners must be in the same world as the door.**).

!!! note "Clock: where the face points"
    For a `clock`, the face of the clock points toward you when you click the second corner. Stand in front
    of the clock when you click it.

### 4. Pivot

**Click the block of the door the rotation passes through (the pivot).** Chat shows
**Pivot set at x, y, z.**

The block must be inside the region (**The pivot must be a block inside the selected region.**). Each type has
its own rule:

| Type | Where the pivot goes |
|---|---|
| `big_door` | Any block of the region. The door turns around the vertical line through it. |
| `drawbridge` | A block on an edge of the region: the hinge row the bridge turns around. |
| `folding_door`, `flag` | A block on one of the two vertical edges of the panel. |

### 5. Pivot axis (drawbridge only)

If you click a corner block as the pivot of a drawbridge that is not a flat wall, the hinge row could run in
two directions. Chat shows
**The pivot axis is ambiguous: click a second block on the pivot row.**

Click another block of the hinge row, at the same height as the pivot. Chat shows **Pivot axis set.**

### 6. Direction

**Choose the opening direction:** followed by one button per valid direction. The current choice is green and
bold. When you arrive at this step, the type default is already selected and the preview already shows where
the door goes (see [Preview](#preview)). Click a direction to choose it and continue; chat shows
**Direction: ...**

### 7. Parameters

**Choose ... (current: ...):** followed by the suggested values, **[Other value]** and **[Continue]**.

- Click a value to choose it. Chat shows the new value and the next parameter (or the next step) follows.
- **[Continue]** keeps the current value.
- **[Other value]** asks **Type the value of ... in chat.** An invalid value shows
  **Invalid value: ...** and asks again. Typing `cancel` returns to the buttons.

| Type | Parameter | Values |
|---|---|---|
| `split_door` | **Split mode** | **Two halves (side by side)**, **Two halves (stacked)**, **Four quadrants (iris)** |
| `rolling_shutter` | **Visible rows** | 0 up to the number of rows minus 1 |

The other type settings are changed after the creation from the **Type settings** menu or with `/dp set`.
See [Editing and deleting](editing.md#type-settings).

### 8. Triggers

**Add triggers to open the door (optional). Collected: N.** with **[Add trigger]**, **[Done]**, **[Back]** and
**[Cancel]**.

The door does not exist yet, so the triggers you add here are collected and linked right after the door is
created. To add one:

1. Click **[Add trigger]**. Chat shows **What do you want to use as trigger?** with **[Block]**,
   **[Redstone]**, **[Entity]**, **[Proximity]** and **[Back]**.
2. Choose the kind:
    - **[Block]**: click a block (button, lever, pressure plate, any block, or a custom block).
    - **[Redstone]**: click a block that will receive redstone power.
    - **[Entity]**: click an entity, a furniture or an NPC.
    - **[Proximity]**: type the radius in chat (1 to `limits.max-proximity-radius`, default 16), then click
      **[Door centre]** or **[My position]** to place the centre of the zone.
3. For Block, Redstone and Entity you have `triggers.selection-timeout-seconds` (default 30) seconds to click.
   Chat then shows **Captured ... at x, y, z. It will be linked when the door is created.**
4. The step comes back with the new count. Add more, or click **[Done]**.

Each click on **[Add trigger]** collects one trigger. Clicking the same source twice shows
**That source has already been captured.** Block and Entity triggers are created with the action Toggle;
Redstone and Proximity triggers with Follow state. You need the permission for each kind of trigger
(`doorplus.trigger.<kind>`, given to every player by default).

After the door is created, chat shows **Triggers linked: N/M.** If one cannot be linked (too far from the door,
inside the door, trigger limit reached and so on), chat shows **Could not link the ... trigger at x, y, z:** and
the reason. The door is created anyway. See [Triggers](../access/triggers.md).

### 9. Confirm

**Check the summary and confirm:** followed by the type, name, world, size, pivot, direction, parameters and
number of triggers, and the buttons **[Create]**, **[Back]** and **[Cancel]**.

Click **[Create]**. Door+ runs the [checks below](#validation-errors). If they pass, chat shows
**Door ... created (#id, N blocks).**, the stick is removed and the collected triggers are linked.

If the destination of the door is blocked, you also see
**Warning: the destination of the door is obstructed. It will not move until the space is free.** The door is
created anyway.

A new door starts with the values of the `defaults` section of `config.yml`: privacy **Private**, redstone off,
protected blocks off, auto-close off, region access off.

## Preview

While you create a door you see particle outlines that only you can see. They are redrawn every half second
and only when you are within about 128 blocks.

| Colour | Shows |
|---|---|
| Light blue | The selected region (after the first corner: that block) |
| Orange | The pivot, after the pivot step |
| Green | Where the door goes when it moves, once the direction is known |
| Red | Same as green, but something is in the way |

## Ending a creation

| Event | Result |
|---|---|
| You click **[Cancel]** or type `/dp cancel` | **Door creation cancelled.** |
| No activity for `creation.timeout-seconds` (default 120) | **Door creation cancelled because of inactivity.** A warning comes 15 seconds before: **The door creation will be cancelled in 15 seconds of inactivity.** |
| You die or change world | **Door creation cancelled.** |
| You leave the server | The creation ends without a message. |
| You click **[Create]** and the door is saved | The creation ends normally. |

Activity means a successful click with the stick, a click on a button or a chat answer. In every case the stick
is removed.

## Validation errors

When you click **[Create]**, Door+ checks the door in the order below. At the first problem it shows the
message and brings you back to the step where you can fix it. The creation stays open.

| Message | Cause | Fix (step you return to) |
|---|---|---|
| **Invalid name ...: use 1-32 letters, digits, _ or -.** | Name missing or not valid | Type a valid name (Name) |
| **You already own a door named ....** | You already have a door with this name | Type another name (Name) |
| **Select both corners first.** | A corner is missing | Click both corners (First corner) |
| **The world of the door is not loaded.** | The world was unloaded | Load the world, then select again (First corner) |
| **Doors cannot be created in the world ....** | The world is in `worlds.disabled` | Build in another world (First corner) |
| **The region goes beyond the height limits of the world.** | The region is above or below the build limits | Select a lower or higher region (First corner) |
| Shape error of the door type | The region, pivot, direction or parameter does not fit the type | See [Shape errors](#shape-errors) |
| **The region has more than N blocks, the most you can use for a door.** | More door blocks than your size limit (`limits.max-door-blocks`, default 500) | Select a smaller region, or ask for a higher limit (First corner) |
| **The region contains no blocks.** | The region holds only air or replaceable blocks | Build the door first (First corner) |
| **The block ... at x, y, z cannot be part of a door.** | A blacklisted block, a block not in the whitelist, or a block with extra data such as a chest or a sign | Remove that block or select around it (First corner) |
| **The region overlaps the door ....** | The region touches a block of another door | Select a region that does not touch other doors (First corner) |
| **You cannot create a door here (denied by ...).** | A protection plugin (for example WorldGuard) does not allow you to build in the region or where the door moves | Build where you are allowed, or ask staff (First corner) |
| **The creation was cancelled by another plugin.** | Another plugin blocked the creation | Ask staff (Confirm) |
| **You already own the maximum number of doors (N).** | You reached your door limit during the creation | Delete a door or ask for a higher limit (Confirm) |

The blocks that can be part of a door are set in the `blocks` section of `config.yml`. Staff with
`doorplus.admin.bypass.protection` are not stopped by protection plugins; see
[Permissions](../reference/permissions.md).

### Shape errors

Shape errors come from the door type. They can also appear right when you click a pivot.

| Message | Types | Fix (step you return to) |
|---|---|---|
| **Select both corners of the door first.** | All | Click both corners (First corner) |
| **The door must be one block thick.** | `split_door`, `rolling_shutter`, `windmill` | Select a flat region (First corner) |
| **The door must be a vertical wall, one block thick.** | `garage_door`, `folding_door`, `flag` | Select a vertical wall 1 block thick (First corner) |
| **The footprint must be square (now A × B).** | `revolving_door` | Make the region as deep as it is wide (First corner) |
| **The two sides of the face must be equal.** | `windmill` | Make the face square (First corner) |
| **A clock must be 2 blocks thick with a square vertical face.** | `clock` | Select 2 layers with a square face (First corner) |
| **The door must be at least 2 blocks wide.** | `folding_door`, `flag` | Make the panel wider (First corner) |
| **The door is too small for this split mode: each split side needs at least 2 blocks.** | `split_door` | Choose another split mode or a bigger door (First corner) |
| **Click a block of the door to set the pivot.** | Types with a pivot | Click a pivot (Pivot) |
| **The pivot must be a block of the door.** | Types with a pivot | Click a block inside the region (Pivot) |
| **The pivot must be on an edge of the door.** | `drawbridge` | Click a block on the edge of the region (Pivot) |
| **The pivot must be on one of the two vertical edges of the door.** | `folding_door`, `flag` | Click a block on the left or right edge (Pivot) |
| **The pivot axis is not set: click a second block on the pivot row.** | `drawbridge` | Click a second block (Pivot axis) |
| **Click another block of the pivot row, at the same height as the pivot.** | `drawbridge` | Click a block in line with the pivot, same height (Pivot axis) |
| **This direction is not valid for this door.** | Types with a direction | Choose another direction (Direction) |
| **Visible rows must be between 0 and N.** | `rolling_shutter` | Choose a smaller value (Parameters) |
| **Unknown split mode.** | `split_door` | Choose a split mode (Parameters) |

## Redefining the region of an existing door

`/dp redefine <door>` (or **Redefine region** in the door menu) runs the corner, pivot, pivot axis and
direction steps again on an existing door, then shows the summary with **[Apply]** instead of **[Create]**.
The access list, triggers and settings are kept. The rules, checks and stick are the same as for a new door,
except that the name is not asked, there are no parameter and trigger steps, and the new region must be in
the same world. See [Editing and deleting](editing.md#redefine-the-region).
