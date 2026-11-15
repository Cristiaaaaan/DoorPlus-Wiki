# Editing and deleting

This page explains how to change a door after it is created: its settings, name, region, owner, and how to
delete it. It also explains how to name a door in commands. It is for door owners, door admins and staff.

Every change can be made from the door menu or by command. The door menu is described button by button on the
[Menus](menus.md#door-menu) page.

## Who can change what

You need a role on the door (see [Privacy and roles](../access/privacy-and-roles.md)) and, for commands, the
permission `doorplus.command.manage` (every player has it by default).

| Change | Minimum role |
|---|---|
| Lock / unlock, rename, redefine, all `/dp set` options, type settings | Admin |
| Privacy, access list, transfer, delete | Owner |

Staff with `doorplus.admin.bypass.edit` act as Admin on every door; with `doorplus.admin.bypass.manage` they act
as Owner. The console always acts as Owner. Without the role you get **You need to be an admin of that door.**

## Referring to a door in commands

Wherever a command asks for `<door>`, you can write:

| Form | Example | Meaning |
|---|---|---|
| `<name>` | `frontgate` | Your own door with that name. Case does not matter. |
| `<owner>:<name>` | `Steve:frontgate` | The door `frontgate` of the player `Steve`. Use it for doors shared with you. |
| `#<id>` | `#12` | The door with that id. The id is shown by `/dp list`, `/dp info` and the menus. |

- Tab completion suggests your own doors first, then `owner:name` for the doors shared with you.
- You can only refer to doors on which you have a role (or with `doorplus.admin.bypass.use`). Any other door
  gives **Door not found.**
- The console, and players with `doorplus.admin.bypass.use`, `doorplus.admin.bypass.edit` or
  `doorplus.admin.bypass.manage`, can also use a bare name for a door of another player, when only one door on
  the server has that name. If several doors share it, chat lists them:
  **More doors are called ...: #4 (Steve), #9 (Alex). Use #id.**

## Door settings

=== "Menu"

    Open the door menu with `/dp menu <door>`, or left-click the door in the main menu (`/dp`). Each setting
    is a button; the [Menus](menus.md#door-menu) page lists the clicks for each one.

=== "Command"

    Type `/dp set <door> <option> <value>`. Chat confirms with **... of ... set to ....**
    Tab completion lists the options of that door and suggested values.

| Option | Values | Default | Notes |
|---|---|---|---|
| `direction` | A direction valid for the door: `cw`, `ccw`, `up`, `down`, `north`, `east`, `south`, `west` | Chosen at creation | Door must be closed and still. Tab completion shows only the valid ones. |
| `blocks-to-move` | `0` to `limits.max-blocks-to-move` (default 64) | `0` = type default | Only `portcullis`, `sliding_door`, `split_door`. Door must be closed and still. |
| `auto-close` | Seconds, `0` to `limits.max-auto-close-seconds` (default 600) | `defaults.auto-close-seconds` (0) | `0` turns auto-close off. |
| `speed` | Multiplier between `limits.speed-multiplier.min` and `.max` (default 0.25 to 4.0) | 1.0 | `2` moves twice as fast. |
| `collision` | `default`, `none`, `shulker`, `push` (only the modes in `collision.allowed-modes`) | `default` (uses `collision.default-mode`) | Shown as **None**, **Solid**, **Push**. See [Movement and animation](movement.md). |
| `redstone` | `true` / `false` (also `on`/`off`, `yes`/`no`) | `defaults.allow-redstone` (false) | Allows Redstone triggers on this door. |
| `protect` | `true` / `false` | `defaults.protect-blocks` (false) | Only admins can break the door blocks and its trigger sources. |
| `region-access` | `true` / `false` | `defaults.region-access` (false) | Members of the WorldGuard region around the door can use it; the door must be Restricted. See [WorldGuard](../integrations/worldguard.md). |
| `base-state` | No value | – | Makes the current position the closed position. See [below](#set-the-closed-position). |
| Type setting | Depends on the type | Depends on the type | See [Type settings](#type-settings). |

Values out of range give **Value out of the allowed range.** or **Invalid value: ....** Changing the direction,
blocks to move or a geometric type setting while the door is open gives
**Close the door first: this can only change while the door is closed and still.**; while it moves,
**The door is moving. Try again when it stops.**

Privacy is not a `/dp set` option: use `/dp privacy <door> <private|restricted|public>`
(Owner only). See [Privacy and roles](../access/privacy-and-roles.md).

### Type settings

Some door types have their own settings. Change them with `/dp set <door> <setting> <value>` or in the
**Type settings** menu of the door. The settings marked "closed" can change only while the door is closed and
still.

| Type | Setting | Closed | Details |
|---|---|---|---|
| `split_door` | `mode` | Yes | [Split door](types/split-door.md) |
| `rolling_shutter` | `visible-rows` | Yes | [Rolling shutter](types/rolling-shutter.md) |
| `platform` | `stops` | Yes | [Platform](types/platform.md) |
| `revolving_door` | `quarter-turns` | No | [Revolving door](types/revolving-door.md) |
| `windmill` | `degrees-per-second` | No | [Windmill](types/windmill.md) |
| `clock` | `time-source` | No | [Clock](types/clock.md) |

The other types have no type settings: their **Type settings** button shows **Not available for this door
type.** An option the door does not have gives **Unknown option ... for this door.**

## Lock and unlock

A locked door cannot be operated by anyone until it is unlocked.

=== "Menu"

    Click **Lock** in the door menu. The button glows while the door is locked.

=== "Command"

    `/dp lock <door>` → **Door ... locked.**
    `/dp unlock <door>` → **Door ... unlocked.**

## Rename

=== "Menu"

    1. Click **Rename** in the door menu. The menu closes.
    2. Chat shows **Type the new name for ... in chat.**
    3. Type the new name. An invalid or taken name shows the error and asks again. Type `cancel` to keep the
       old name.

=== "Command"

    `/dp rename <door> <name>` → **Door renamed to ....**

Name rules are the same as at creation: 1 to 32 letters, digits, `_` or `-`, unique among the doors of the same
owner, compared without case. You can change only the case of the name (`gate` → `Gate`). A door can be renamed
in any state. Errors: **Invalid name: use 1-32 letters, digits, _ or -.** and **That name is already in use.**

## Redefine the region

Redefining selects the blocks of the door again, for example after you made the door bigger. The access list,
triggers and settings are kept.

1. Close the door and wait until it stops. Otherwise you get **Close the door first...** or
   **The door is moving...**.
2. Type `/dp redefine <door>`, or click **Redefine region** in the door menu.
3. You receive the creation stick and **Redefining the region of .... Use the stick you just received.**
4. Click the two corners, then the pivot, the pivot axis and the direction when the type has them.
5. Check the summary and click **[Apply]**.
6. Chat shows **The region of ... has been redefined.**

The new region must be in the same world as the door. It goes through the same checks as a new door (size
limit, blocks that can be animated, overlap with other doors, protection plugins); see
[Creating a door](creating.md#validation-errors). You need a free inventory slot, and you cannot redefine while
you are creating another door.

## Set the closed position

Door+ remembers which position of a door is "closed". **Set as closed state** in the menu, or
`/dp set <door> base-state`, makes the current position the closed one. It is useful after an import, or to swap
the meaning of open and closed.

- If the door is open, its opening direction is reversed, so that opening it brings it back to the old closed
  position.
- It works only for big doors, drawbridges, portcullises, sliding doors, garage doors, split doors and folding
  doors whose current position is a single box. Other cases give **This door does not support that.**
- The door must not be moving.

Chat shows **The current position of ... is now its closed state.**

## Transfer to another player

Only the owner can transfer a door.

=== "Menu"

    1. Click **Transfer** in the door menu. The menu **Choose a player** opens with the online players.
    2. Click a head, or click **Enter a name** and type the name of a player who is offline.
    3. The menu **Are you sure?** asks **Transfer ... to ...?**. Click **Confirm**.

=== "Command"

    `/dp transfer <door> <player>`

Chat shows **Door ... transferred to ....**

- The new owner must not already own a door with the same name (**That name is already in use.**).
- The new owner must have room in their door limit (**That player has reached their door limit.**). For an
  offline player the limit in `limits.doors-per-player` is used. The check is skipped for the console and for
  players with `doorplus.admin.bypass.limits` using the command.
- If the new owner was in the access list, that entry is removed: the owner already has full access.
- You lose ownership of the door.

## Take ownership as staff

`/dp admin claim <door>` makes you the owner of any door. It needs `doorplus.admin.claim` (default op) and must
be run by a player.

- `<door>` can be `#id`, `owner:name`, one of your door names, or a name that only one door on the server has.
- Chat shows **You are now the owner of ... #id (it belonged to ...).**
- If you already own it: **You already own ....**
- The door limit is not checked. If you already own a door with the same name, the claim fails with
  **That name is already in use.**; rename one of them first.

## Teleport to a door

`/dp tp <door>` teleports you next to the door, facing it. **Teleport** in the door menu does the same but looks
for a free, safe spot around the door first; if there is none, chat shows **No free spot found next to ....**

Both need the permission `doorplus.command.teleport` (default op) and a role on the door. If the world of the
door is not loaded, chat shows **The world ... of ... is not loaded.**

## Delete

Only the owner can delete a door.

=== "Menu"

    1. Click **Delete** in the door menu.
    2. The menu **Are you sure?** asks **Delete ... (#id)?** Click **Confirm**, or **Cancel** to go back.
    3. Chat shows **Door ... deleted.** and the main menu opens.

=== "Command"

    1. Type `/dp delete <door>`.
    2. Chat shows **Delete ... for good? Type /door delete #id confirm to confirm.**
    3. Type `/dp delete <door> confirm`.
    4. Chat shows **Door ... deleted.**

What happens:

- The blocks stay where they are, in the current position of the door.
- The access entries, the triggers of the door and its group memberships are removed.
- A moving door cannot be deleted (**The door is moving. Try again when it stops.**). Windmills, flags and clocks
  are stopped first.
- An open rolling shutter is closed instantly first. If something is in the way, the deletion is refused with
  **The door cannot close: something is in the way.**

Deleting cannot be undone.

## See the details of a door

`/dp info <door>` shows the type, state, owner, your role, location, size, number of blocks, direction, privacy,
locked state, redstone, protected blocks, region access, auto-close, speed, collisions, key mode, access entries,
number of triggers and how many times the door moved.

`/dp list` lists your doors and, below, the doors shared with you. `/dp list <player>` lists the doors of another
player and needs `doorplus.admin.list`. The full list of commands is on the
[Commands](../reference/commands.md) page.
