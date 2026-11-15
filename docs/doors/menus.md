# Menus

This page is a tour of every Door+ inventory menu: how to open it, what each button does, and which click to
use. It is for every player who manages doors from the menus instead of commands.

## How menus work

- Every menu is a large chest (6 rows). The bottom row holds **Back** (left corner, not in the main menu),
  **Close** (centre) and, when a list has more than one page, **Previous page** and **Next page** with the page
  number.
- You cannot take items out of a menu. Clicks act one tick after you click; each click plays a short sound
  (`gui.click-sound` in `config.yml`).
- A button you cannot use stays visible: its description ends with the reason in red and clicking it does
  nothing. The reasons are:

| Reason shown | Meaning |
|---|---|
| **Requires role ... on this door.** | You need that role (User, Admin or Owner) on the door. |
| **You do not have permission for this.** | You lack the permission for that action. |
| **Not available while the door is moving.** | Wait until the door stops. |
| **Close the door first (state 0).** | The change is allowed only while the door is closed. |
| **Not available for this door type.** | The door type does not use this setting. |
| **Only the owner of the group can do this.** | The group belongs to another player. |

- Menus update by themselves when the door changes. If the door you are looking at is deleted, the menu closes
  with **The door you were editing has been deleted.** (for groups: **The group you were editing has been
  deleted.**).

### Click types

Where a button says nothing else, any click works. Number buttons use:

| Click | Effect |
|---|---|
| Left click | Increase by one step / next value |
| Right click | Decrease by one step / previous value |
| ++shift++ + left or right click | Ten steps at once (where the description says **Shift: ×10**) |

### Chat input

Some buttons ask you to type text in chat (a name, a search text, a permission node, a value).

1. The menu closes and chat shows the question and **Type cancel in chat to abort.**
2. Type the answer. Other players do not see it.
3. If the answer is not valid, chat shows why and asks again.
4. Type `cancel` (or `/dp cancel`) to abort: chat shows **Input cancelled.** and the menu opens again.
5. After 30 seconds without an answer, chat shows **Input timed out.** and the menu opens again.

## Main menu

Open it with `/dp` or `/dp menu` (permission `doorplus.command.menu`, every player has it by default). Title:
**Door+ — Your doors**.

The top five rows list the doors, one icon per door, with ID, type, state, privacy, owner and your role.

| Click on a door | Effect |
|---|---|
| Left click | Opens the [door menu](#door-menu) |
| Right click | Opens or closes the door |

Bottom row:

| Button | Effect |
|---|---|
| **Filter** | Click: next filter. **My doors** → **Shared with me** → **All doors** (only with `doorplus.admin.list`) → back. "Shared with me" lists the doors of other players on which you have a role. |
| **Sort** | Left click: next order, right click: previous. Orders: name, type, creation date, distance. |
| **Search** | Left click: type part of a door name in chat (**Type part of the door name in chat.**, at most 32 characters). Right click: clear the search. |
| **Create a door** | Opens **Choose the door type** with the enabled types you may create. Click a type to start the creation. If you can create none, the button shows **You cannot create any door type.** See [Creating a door](creating.md). |
| **Door groups** | Opens [Your door groups](#your-door-groups). |

## Door menu

Open it by left-clicking a door in the main menu, or with `/dp menu <door>`. Title: **Door** followed by the
door name.

### Second row: state and access

| Button | Role | Effect |
|---|---|---|
| Door name and `#id` | – | Information only: type, state, owner, world and position, size and blocks, creation date, number of movements, last movement and who made it. |
| **Open / Close** | User | Click: open or close the door. For a platform: left click goes to the next stop, right click to the previous one. |
| **Lock** | Admin | Click: lock or unlock. A locked door cannot be operated by anyone. The button glows when locked. |
| **Privacy: ...** | Owner | Left click: Private → Restricted → Public → Private. Right click: opens the [access list](#access-list). The description shows the meaning of the current privacy and the number of access entries. |
| **Access list** | Owner | Opens the [access list](#access-list). |
| **Triggers** | Admin | Opens the [triggers menu](#triggers-menu). Shows the number of triggers. |
| **Key** | Admin | Opens the [key menu](#key-menu). Shows the key mode. |

### Third row: movement settings

| Button | Role | Effect |
|---|---|---|
| **Auto-close** | Admin | Left: +1 s, right: −1 s, shift: ×10. From **off** (0) to `limits.max-auto-close-seconds` (600). |
| **Speed** | Admin | Left: +0.25, right: −0.25, within `limits.speed-multiplier` (0.25 to 4.0). |
| **Direction** | Admin | Left: next valid direction, right: previous. Door closed and still. |
| **Blocks to move** | Admin | Left: +1, right: −1, shift: ×10, up to `limits.max-blocks-to-move` (64). **type default** means 0. Only portcullis, sliding door and split door; door closed and still. |
| **Collisions** | Admin | Left: next mode, right: previous. **default (...)** follows `collision.default-mode`; the others are the modes allowed in `collision.allowed-modes` (**None**, **Solid**, **Push**). |
| **Redstone** | Admin | Click: allow or forbid Redstone triggers. If redstone is off server-wide, the description says **Redstone is disabled on this server.** |
| **Protect blocks** | Admin | Click: on or off. When on, only admins can break the door blocks and trigger sources. |
| **Region access** | Admin | Click: on or off. Members and owners of the WorldGuard region around the door can use it; the door must be Restricted. Without WorldGuard the description says **WorldGuard is not active on this server.** |

A setting button glows while it is on. The meaning of each setting is explained in
[Editing and deleting](editing.md#door-settings).

### Fourth row: management

| Button | Role | Effect |
|---|---|---|
| **Type settings** | Admin | Opens the [type settings menu](#type-settings-menu). Not available for types without settings. |
| **Groups** | Admin | Opens the [groups of the door](#groups-of-a-door). |
| **Rename** | Admin | Chat input: **Type the new name for ... in chat.** An invalid or taken name asks again. |
| **Redefine region** | Admin | Closes the menu and starts the redefinition with the creation stick. Door closed and still. See [Redefine the region](editing.md#redefine-the-region). |
| **Set as closed state** | Admin | The current position becomes the closed position. Not while moving. See [Set the closed position](editing.md#set-the-closed-position). |
| **Teleport** | User + `doorplus.command.teleport` | Teleports you to a free spot next to the door. |
| **Transfer** | Owner | Choose a player, then confirm. See [Transfer](editing.md#transfer-to-another-player). |
| **Delete** | Owner | Asks for confirmation, then deletes the door. See [Delete](editing.md#delete). |

## Confirmation menu

Title: **Are you sure?** It opens before deleting a door or a group, transferring a door, regenerating a key,
removing an Admin from the access list and removing a trigger.

- The middle icon describes what will happen.
- **Confirm** does it. It cannot be undone.
- **Cancel** (or **Back**) returns to the previous menu without changes.

## Access list

Open it from **Access list** or by right-clicking **Privacy** in the door menu. Owner only. Title:
**Access —** followed by the door name.

Top row:

| Button | Effect |
|---|---|
| **Add player** | Opens [Choose a player](#pickers). |
| **Add LuckPerms group** | Opens **Choose a LuckPerms group**. Needs LuckPerms and `doorplus.access.group`. |
| **Add permission** | Chat input: **Type the permission node in chat (e.g. myserver.vip).** Needs `doorplus.access.permission`. |

After you pick a subject, the [role menu](#role-menu) asks which role to give.

The rows below list the entries (players, then groups, then permissions) with type, role, who added them and
when.

| Click on an entry | Effect |
|---|---|
| Left click | Switches the role between User and Admin |
| ++shift++ + right click | Removes the entry. Removing an Admin asks for confirmation first. |

A group entry whose LuckPerms group no longer exists shows **This LuckPerms group no longer exists.** See
[Privacy and roles](../access/privacy-and-roles.md).

## Role menu

Title: **Role for** followed by the subject. It opens after you choose a player, group or permission to add.

1. Click **User** or **Admin**. User is selected at the start; the selected role glows and says **Selected**.
2. Click **Confirm**. Chat shows **... now has role ... on ....** and the access list opens again.

## Pickers

Pickers are lists to choose from. They all have a **Search** button at the top: left click to type a text in
chat (**Type the text to search for in chat.**), right click to clear it. An empty list shows
**Nothing to show.**

| Picker | Lists | Extra button |
|---|---|---|
| **Choose a player** | Online players you can see, as heads | **Enter a name**: type the name of an offline player in chat. An unknown name shows **Unknown player ....** |
| **Choose a LuckPerms group** | The groups loaded in LuckPerms | – |
| **Choose the door type** | The enabled types you may create | – |
| **Choose a door** | Doors where you are Admin or Owner, not yet in the group | – |
| **Choose one of your groups** | Your groups that do not contain the door yet | – |

Click an entry to choose it.

## Triggers menu

Open it from **Triggers** in the door menu (Admin) or in a group menu (group owner). Title: **Triggers —**
followed by the door or group name.

| Button | Effect |
|---|---|
| **Add trigger** | Opens **New trigger —** with the four kinds below. |
| **Inspect** | Closes the menu. Click a block or entity within the selection time to list the triggers linked to it. |
| A trigger | Click: opens the [trigger edit menu](#trigger-edit-menu). The list shows id, type, provider, position, action, access and whether it is enabled. Enabled triggers glow. |

In **New trigger**, choose:

| Button | What you click next | Permission (any of) |
|---|---|---|
| **Block** | A button, lever, plate, any block or a custom block | `doorplus.trigger.block`, `doorplus.trigger.custom_block` |
| **Redstone** | A block that receives redstone power | `doorplus.trigger.redstone` |
| **Entity** | An entity, a furniture or an NPC | `doorplus.trigger.entity`, `.furniture`, `.npc` |
| **Proximity** | Nothing: you type the radius in chat and choose the centre | `doorplus.trigger.proximity` |

The menu closes while you select the source. When you finish, the triggers menu opens again. See
[Triggers](../access/triggers.md).

## Trigger edit menu

Title: **Trigger #id —** followed by the type. Changes are saved at once.

| Button | Shown for | Effect |
|---|---|---|
| **Action** | All | Left: next action, right: previous. Only the actions valid for this source and target are offered. |
| **Stop** | Action "Go to stop" | Left: +1, right: −1, from 0 to the last stop of the platform. |
| **Access** | All except Redstone | Left: next mode, right: previous (Door privacy, Public, Restricted, Owner and admins). When the mode is Restricted, right click opens the [own entries](#own-entries-of-a-trigger) instead. |
| **Cooldown** | All | Ticks between two activations. Left: +1, right: −1, shift: ×10. |
| **Cancel vanilla interaction** | Triggers activated by a player, except Proximity | Click: on or off. |
| **Enabled** | All | Click: enable or disable the trigger. |
| **Radius** | Proximity | Left: +1, right: −1, from 1 to `limits.max-proximity-radius` (16). |
| **Exit delay** | Proximity | Ticks. Left: +1, right: −1, shift: ×10. |
| **Highlight** | All | Closes the menu and shows particles on the source for 10 seconds, only to you. |
| **Remove** | All | Asks for confirmation, then removes the trigger. |

The meaning of each option is explained in [Triggers](../access/triggers.md).

## Own entries of a trigger

Title: **Own entries — trigger #id**. Open it by right-clicking **Access** in the trigger edit menu when the
access mode is Restricted.

It works like the [access list](#access-list), without roles: **Add player**, **Add LuckPerms group** and
**Add permission** at the top add an entry directly; ++shift++ + right click on an entry removes it. With no
entries, only the owner and the admins can use the trigger.

## Type settings menu

Open it from **Type settings** in the door menu (Admin). Title: the type name, **settings —** and the door name.

There is one button per setting of the type. Two kinds of button exist:

- **Cycle buttons**: left click next value, right click previous, ++shift++ ×10.
- **Text buttons** (the platform stops): click, then type the value in chat. The prompt shows the current value
  and examples. An invalid value shows the error and asks again.

Settings that change the shape of the door are available only while the door is closed and still. The settings
of each type are listed in [Editing and deleting](editing.md#type-settings).

## Key menu

Open it from **Key** in the door menu (Admin). Title: **Key —** followed by the door name.

| Button | Effect |
|---|---|
| **Key of ...** | Information: mode, key, and whether a key is consumed on use. |
| **Mode** | Left: next mode, right: previous. **No key**, **Key as alternative**, **Key required**. |
| **Create from the item in hand** | Marks the item in your main hand as a key of this door. |
| **Use the custom item in hand** | Any item with the same custom id (ItemsAdder, Nexo, ...) opens the door. |
| **Give a copy** | Gives you a new key item. |
| **Consume on use** | Click: on or off. |
| **Regenerate** | Asks for confirmation, then makes a new key: old keys stop working. |
| **Remove the key** | Removes the key; the mode goes back to No key. |

See [Keys](../access/keys.md).

## Your door groups

Open it from **Door groups** in the main menu. Title: **Your door groups**. It lists the groups you own.

| Button | Effect |
|---|---|
| **Create group** | Chat input: **Type the name of the new group in chat.** The new group opens. An invalid or taken name asks again. |
| A group | Click: opens the [group menu](#group-menu). |

## Group menu

Title: **Group** followed by the group name. Only the owner of the group (or staff with
`doorplus.admin.bypass.manage`) can use the buttons.

| Button | Effect |
|---|---|
| Group name | Information: owner and number of doors out of `limits.max-doors-per-group`. |
| **Add door** | Opens **Choose a door**. |
| **Open / Close all** | Left click: open every door. Right click: close every door. |
| **Triggers** | Opens the [triggers menu](#triggers-menu) of the group. |
| **Rename** | Chat input: **Type the new name for ... in chat.** |
| **Delete group** | Asks for confirmation. The doors are kept; triggers targeting the group are removed. |

| Click on a member door | Effect |
|---|---|
| Left click | Opens its door menu |
| Right click | Removes it from the group |

See [Door groups](../access/groups.md).

## Groups of a door

Open it from **Groups** in the door menu (Admin). Title: **Groups of** followed by the door name.

| Button | Effect |
|---|---|
| **Add to a group** | Opens **Choose one of your groups**. Click a group to add the door. |
| A group | Left click: opens the group menu (if you can manage it). Right click: removes the door from the group. |
