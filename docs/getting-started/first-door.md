# Your first door

This tutorial walks you through building a simple big door, opening it, linking a lever, making it public and
making it close by itself. It is for any player who can create doors; it takes about five minutes.

Before you start, you need the permission `doorplus.create.big_door` (operators have it). See
[Installation](installation.md#let-players-create-doors).

## 1. Build the door

1. Choose a wall with an opening 3 blocks wide and 4 blocks high.
2. Fill the opening with a solid block, for example oak planks. The door is a flat panel: 3 wide, 4 high
   and 1 block thick.
3. Leave free space on one side of the wall, at least 3 blocks deep. The door swings into that space.

!!! warning "Blocks that cannot be part of a door"
    By default, blocks with extra data (chests, signs, furnaces, banners and similar) and blocks such as
    bedrock or command blocks cannot be part of a door. Use plain building blocks for this tutorial.

## 2. Start the creation

1. Empty one slot of your inventory, ideally the slot in your hand.
2. Type `/dp create big_door frontgate`.
   `frontgate` is the name of the door: 1 to 32 letters, digits, `_` or `-`.
3. You receive the **Door+ creation stick** and this message in chat:
   **Creating a Big door. Use the stick you just received.**

Each instruction starts with the step number, for example **[1/6]**. Under most instructions there are
clickable chat buttons such as **[Back]** and **[Cancel]**.

## 3. Select the region

1. Read the instruction **[1/6] Click the first corner of the door with the stick.**
2. Hold the stick and click the bottom-left block of the panel. Left or right click both work, and the
   stick never breaks blocks. Chat shows **First corner set at x, y, z.**
3. Read **[2/6] Click the opposite corner of the door.**
4. Click the top-right block of the panel, diagonally opposite to the first one. Chat shows
   **Second corner set at x, y, z. Region: 3x4x1** (or `1x4x3`, depending on how the wall is oriented).

A light blue outline of particles now shows the selected region. Only you can see it.

## 4. Choose the pivot

1. Read **[3/6] Click the block of the door the rotation passes through (the pivot).**
2. Click the bottom block of the column where the hinges should be (the left or the right edge of the
   panel). Chat shows **Pivot set at x, y, z.**

The door will turn around the vertical line that passes through this block. The pivot block is outlined in
orange.

## 5. Choose the direction

1. Read **[4/6] Choose the opening direction:** with the buttons **[Clockwise]** and
   **[Counter-clockwise]**. The highlighted one (green, bold) is the current choice.
2. Look at the green outline: it shows where the door will be when it is open, with the highlighted
   direction. If the outline is red, something is in the way.
3. Click the direction you want. Chat shows **Direction: Clockwise** (or the other one).

If the door ends up opening the wrong way, click **[Back]** to return to this step.

## 6. Skip the triggers for now

1. Read **[5/6] Add triggers to open the door (optional). Collected: 0.**
2. Click **[Done]**. You will add a lever later in this tutorial.

## 7. Confirm

1. Read **[6/6] Check the summary and confirm:** and the summary: type, name, world, size, pivot,
   direction, parameters and triggers.
2. Click **[Create]**.
3. Chat shows **Door frontgate created (#1, 12 blocks).** The number after `#` is the door id.
   The creation stick disappears from your inventory.

If something is wrong, chat shows the reason and takes you back to the step to fix. See
[Creating a door](../doors/creating.md#validation-errors) for every error.

## 8. Open and close the door

=== "Command"

    1. Type `/dp toggle frontgate`. The door swings open.
    2. Type `/dp toggle frontgate` again. The door closes.

    `/dp open frontgate` and `/dp close frontgate` only open or only close.

=== "Menu"

    1. Type `/dp`. The menu **Door+ — Your doors** opens and shows your doors.
    2. Right-click the icon of `frontgate`. The door opens or closes.
    3. Left-click the icon to open the door menu, then click **Open / Close**.

## 9. Add a lever

1. Place a lever on a block next to the door, but not on the door itself (a trigger cannot move with the
   door).
2. Type `/dp menu frontgate`. The menu **Door frontgate** opens.
3. Click **Triggers**, then **Add trigger**. The menu **New trigger — frontgate** opens.
4. Click **Block**. The menu closes and chat shows
   **Click the block to use as a trigger. You have 30 seconds.**
5. Click the lever. Chat shows **Linked Block trigger #1 to frontgate.** and the buttons
   **[Add another]** and **[Done]**.
6. Click **[Done]**. The triggers menu opens again and lists the lever.
7. Flip the lever. The door opens or closes.

You can do the same with `/dp trigger add frontgate block`, and you can also collect triggers during the
creation (step 6). See [Triggers](../access/triggers.md) for every kind of trigger.

## 10. Make the door public

A new door is **Private**: only you (and the admins you add) can use it, also through the lever.

=== "Menu"

    1. In the menu **Door frontgate**, read the button **Privacy: Private**.
    2. Left-click it twice: Private → Restricted → Public.

=== "Command"

    1. Type `/dp privacy frontgate public`.
    2. Chat shows **Privacy of frontgate set to Public.**

Now every player can open the door with the lever. See [Privacy and roles](../access/privacy-and-roles.md)
for the Restricted mode and the access list.

## 11. Close the door automatically

=== "Menu"

    1. In the menu **Door frontgate**, find **Auto-close** (value **off**).
    2. Shift + left-click it once (+10 seconds). The value becomes **10 s**.

=== "Command"

    1. Type `/dp set frontgate auto-close 10`.
    2. Chat shows **auto-close of frontgate set to 10.**

The door now closes 10 seconds after it opens. Set the value back to `0` to turn auto-close off. See
[Movement and animation](../doors/movement.md) for how auto-close behaves.

## Next steps

- [Creating a door](../doors/creating.md): every step of the creation for every door type.
- [Door types](../doors/types/index.md): the 13 types and what each one does.
- [Editing and deleting](../doors/editing.md): every setting and command.
- [Menus](../doors/menus.md): every button of every menu.
