# Groups

A door group is a named set of doors that move together, for example the two wings of a castle gate or
all the shutters of a shop. This page explains how to create groups, add and remove doors, operate a group
and link triggers to it. It is for door owners and staff.

## How groups work

- A group has a name, an owner (the player who created it) and a list of doors.
- Group names follow the door name rules: 1 to 32 letters, digits, `_` or `-`. A name is unique among the
  groups of the same owner.
- A door can be in several groups.
- Only the owner of the group manages it: adds doors, operates it, links triggers, renames and deletes it.
  Staff with `doorplus.admin.bypass.manage` can manage any group.
- To add a door to a group you need the role **Admin** or **Owner** on that door. If the group owner later
  loses that role, the door is removed from the group the next time the group is operated.
- Groups are private to their owner: nobody else sees them in menus or can operate them.

## Refer to a group in commands

| Form | Example | Who can use it |
|---|---|---|
| `<name>` | `gates` | The owner of the group. |
| `<owner>:<name>` | `Steve:gates` | Staff with `doorplus.admin.bypass.manage`. |
| `#<id>` | `#4` | The owner, staff with `doorplus.admin.bypass.manage`, and the console (which must use this form or `<owner>:<name>`). |

`/dp group list` shows the id of each group.

## Create a group

=== "Menu"

    1. Run `/dp menu` to open the main menu.
    2. Click **Door groups**. The menu **Your door groups** opens.
    3. Click **Create group**.
    4. Type the name in chat. Chat confirms **Group `<group>` created.** and the group menu opens.

=== "Command"

    ```
    /dp group create <name>
    ```

You can own at most `limits.max-groups-per-player` groups (default `10`). Over the limit: **You have
reached your limit.** `doorplus.admin.bypass.limits` ignores it.

## Add doors

A group holds at most `limits.max-doors-per-group` doors (default `16`). When full: **The group is full.**
`doorplus.admin.bypass.limits` ignores it.

=== "Group menu"

    1. Open the group: **Door groups** → click the group. The menu **Group `<group>`** opens.
    2. Click **Add door**. The list shows the doors where you are admin or owner that are not in the group.
    3. Click a door. Chat confirms: **Door `<door>` added to group `<group>`.**

=== "Door menu"

    1. Open the door menu with `/dp menu <door>`.
    2. Click **Groups**. The menu **Groups of `<door>`** opens.
    3. Click **Add to a group** and choose one of your groups.

=== "Command"

    ```
    /dp group add <group> <door>
    ```

## Remove doors

=== "Group menu"

    In **Group `<group>`**, right click the door. Chat confirms:
    **Door `<door>` removed from group `<group>`.**

=== "Door menu"

    In **Groups of `<door>`**, right click the group. Only the group owner or the **owner** of the door can do
    this; anyone else gets **Only the owner of the group `<group>` or the owner of the door can remove it from the
    group.**

=== "Command"

    ```
    /dp group remove <group> <door>
    ```

!!! note
    In the group menu and with the command, only the group owner can remove doors. The owner of a door can also
    take it out of someone else's group from the door menu **Groups**.

## Operate a group

| Action | Effect |
|---|---|
| Open | Every door moves to open. |
| Close | Every door moves to closed. |
| Toggle | If at least one door is closed, all doors open; otherwise all close. This keeps the doors in sync. |

- Each door goes through its own checks with you as the player: lock, privacy, key and obstacles are
  checked door by door, and refused doors are skipped.
- You get a single message, and only if **every** door was refused.
- A group with no doors: **The group `<group>` has no doors.**

=== "Menu"

    In **Group `<group>`**, click **Open / Close all**: left click opens all, right click closes all.

=== "Command"

    ```
    /dp group toggle|open|close <group>
    ```

    These commands need `doorplus.command.toggle`. Run from the console they skip the access checks and
    report how many doors started moving.

## Triggers on a group

A trigger can target a group: one button opens every door of the group.

=== "Menu"

    1. In **Group `<group>`**, click **Triggers**.
    2. Click **Add trigger** and continue as for a door. See [Link a trigger](triggers.md#link-a-trigger).

=== "Command"

    Write the target as `group:<group>`:

    ```
    /dp trigger add group:gates block
    /dp trigger list group:gates
    ```

- Linking a trigger to a group requires the role Admin on every door of the group, and the source must be
  in the same world and within `limits.max-trigger-distance` of every door.
- A group has at most `limits.max-triggers-per-door` triggers, like a door.
- The trigger action runs on every door as described in [Operate a group](#operate-a-group). **Go to stop**
  runs only on the platforms of the group. The trigger access mode applies to each door.

Details: [Triggers on a group](triggers.md#triggers-on-a-group).

## Rename and delete

=== "Menu"

    In **Group `<group>`**:

    - **Rename**: type the new name in chat.
    - **Delete group**: confirm. The doors are kept; triggers targeting the group are removed.

=== "Command"

    ```
    /dp group rename <group> <name>
    /dp group delete <group>
    ```

## List and inspect groups

```
/dp group list
/dp group info <group>
```

`list` shows your groups with their id and number of doors (the console sees every group). `info` shows the
owner and the doors of a group.

## Commands and permissions

| Command | Permission |
|---|---|
| `/dp group create <name>` | `doorplus.command.manage` |
| `/dp group delete <group>` | `doorplus.command.manage` |
| `/dp group rename <group> <name>` | `doorplus.command.manage` |
| `/dp group add <group> <door>` | `doorplus.command.manage` |
| `/dp group remove <group> <door>` | `doorplus.command.manage` |
| `/dp group list` | `doorplus.command.manage` |
| `/dp group info <group>` | `doorplus.command.manage` |
| `/dp group toggle\|open\|close <group>` | `doorplus.command.toggle` |

Both permissions are in `doorplus.user`, which every player has by default.

## Limits

| Config key | Default | Meaning |
|---|---|---|
| `limits.max-groups-per-player` | `10` | Groups one player can own. |
| `limits.max-doors-per-group` | `16` | Doors in one group. |
| `limits.max-triggers-per-door` | `16` | Triggers per door, and per group. |

`-1` means unlimited. See [Limits](../reference/limits.md).

## Placeholders

PlaceholderAPI placeholders can show group data, such as the number of groups a player owns. See
[Placeholders](../reference/placeholders.md).
