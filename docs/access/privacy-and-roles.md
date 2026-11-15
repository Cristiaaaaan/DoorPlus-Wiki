# Privacy and roles

This page explains who can use a door and who can change it: the door privacy, the three roles, access
entries (players, LuckPerms groups, permissions) and the exact order of checks Door+ runs every time
someone tries to open or close a door. It is for door owners and for server staff.

## Overview

Every door has:

- one **owner**: the player who created it, or the player it was transferred to;
- a **privacy**: `PRIVATE`, `RESTRICTED` or `PUBLIC`;
- an **access list**: entries that give a player, a LuckPerms group or a permission node the role **User**
  or **Admin**;
- optionally a **key** (see [Keys](keys.md)) and **triggers** with their own access mode (see
  [Triggers](triggers.md));
- the **Region access** option (WorldGuard, see [Region access](#region-access)).

New doors start with the privacy in `defaults.privacy` (default `PRIVATE`) and with Region access set by
`defaults.region-access` (default `false`).

## Privacy

| Privacy | Menu name | Who can use the door |
|---|---|---|
| `PRIVATE` | **Private** | Only the owner and the admins. |
| `RESTRICTED` | **Restricted** | The owner, the admins and every access entry (role User included). |
| `PUBLIC` | **Public** | Everyone. |

"Admins" means everyone with the role Admin from the access list. Owner and admins can always use the
door, whatever the privacy, as long as it is not locked.

!!! note
    On a `PUBLIC` door, players without a role use the door through its triggers (buttons, plates,
    proximity zones and so on). Commands such as `/dp toggle` and the door menu only find doors on which
    the player has a role (or a bypass permission).

### Change the privacy

Only the owner can change the privacy.

=== "Menu"

    1. Open the door menu with `/dp menu <door>`.
    2. Left click **Privacy: ...**. The privacy cycles **Private** → **Restricted** → **Public** → **Private**.
    3. Right click the same button to open the access list.

=== "Command"

    ```
    /dp privacy <door> <private|restricted|public>
    ```

    Chat confirms: **Privacy of `<door>` set to `<privacy>`.**

## Roles

A player's role on a door is the highest of:

- **Owner**, if they own the door;
- the role of every access entry that matches them (an entry can be a player, a LuckPerms group or a
  permission node);
- **User**, from [Region access](#region-access), only if no entry gives them a role.

| Action | User | Admin | Owner |
|---|:-:|:-:|:-:|
| Open and close (subject to privacy, key and lock, see [How a use is decided](#how-a-use-is-decided)) | yes | yes | yes |
| See the door in menus and commands, door information, teleport (with `doorplus.command.teleport`) | yes | yes | yes |
| Lock and unlock | | yes | yes |
| Change settings: rename, redefine, base state, auto-close, speed, direction, blocks to move, collision, **Redstone**, **Protect blocks**, **Region access**, type settings | | yes | yes |
| Manage triggers | | yes | yes |
| Manage the key | | yes | yes |
| Add the door to a group, remove it from a group (door menu **Groups**) | | yes | yes |
| Break or change protected blocks and trigger sources (door with block protection on) | | yes | yes |
| Change the privacy | | | yes |
| Add, change and remove access entries | | | yes |
| Transfer and delete the door | | | yes |

Owner and admins never need the key and are never stopped by privacy or trigger access modes.

!!! note
    The roles give the right on the door. To use the commands a player also needs the command permission
    (`doorplus.command.toggle` to operate doors, `doorplus.command.manage` for everything else). Both are
    included in `doorplus.user`, which every player has by default. See
    [Permissions](../reference/permissions.md).

## Access entries

An access entry is a subject with a role, **User** or **Admin**. Entries of different types can be mixed on
the same door.

| Type | Subject | Matches an online player when |
|---|---|---|
| Player | A player (online, or offline but known to the server) | It is that player. |
| Group | The name of a LuckPerms group, lower case | The player has the permission `group.<name>`, which LuckPerms gives to every member of the group (inherited groups included). |
| Permission | A permission node, `a-z 0-9 . _ * -`, up to 128 characters | The player has that permission. |

Rules:

- Adding an entry that already exists changes its role.
- The owner cannot be added to the list: **The owner already has full access and cannot be added to the
  list.**
- A door can have at most `limits.max-access-entries` entries (default `64`). When full: **The access list
  of this door is full.**
- Adding **Group** entries requires LuckPerms and the permission `doorplus.access.group`. Adding
  **Permission** entries requires `doorplus.access.permission`. Both permissions are in `doorplus.user`.
- Group entries saved earlier keep working without LuckPerms through `group.<name>` permissions. In the
  access menu a group that no longer exists in LuckPerms shows **This LuckPerms group no longer exists.**

See [LuckPerms](../integrations/luckperms.md) for how offline players are checked.

### Add an entry

Only the owner can manage access entries.

=== "Menu"

    1. Open the door menu with `/dp menu <door>`.
    2. Click **Access list** (or right click **Privacy: ...**). The menu **Access — `<door>`** opens.
    3. Click one of the buttons at the top:
        - **Add player**: pick an online player, or click **Enter a name** and type the name in chat (for
          players who are offline).
        - **Add LuckPerms group**: pick one of the loaded groups. Disabled with **Requires LuckPerms.**
          if LuckPerms is not installed.
        - **Add permission**: type the permission node in chat, for example `myserver.vip`.
    4. The menu **Role for `<subject>`** opens with **User** selected. Click **Admin** if you want to give
       that role.
    5. Click **Confirm**. Chat confirms: **`<subject>` now has role `<role>` on `<door>`.**

=== "Command"

    ```
    /dp access add <door> <player|group|permission> <subject> [user|admin]
    ```

    The role defaults to `user`. Examples:

    ```
    /dp access add castle player Steve
    /dp access add castle group vip admin
    /dp access add castle permission myserver.shop.use
    ```

### Change the role of an entry

=== "Menu"

    In **Access — `<door>`**, left click the entry. The role switches **User** ↔ **Admin**.

=== "Command"

    Run `/dp access add` again for the same subject with the new role.

### Remove an entry

=== "Menu"

    In **Access — `<door>`**, ++shift+right-button++ the entry. Removing an **Admin** entry asks for
    confirmation first.

=== "Command"

    ```
    /dp access remove <door> <player|group|permission> <subject>
    ```

    Chat confirms: **`<subject>` removed from `<door>`.**

### List the entries

```
/dp access list <door>
```

Shows every entry with its type and role, and the current privacy.

## Region access

With **Region access** on, members and owners of a WorldGuard region that contains the centre of the door
get the role **User** on the door, if they have no higher role from the access list.

- It works like any other User entry: the door must be `RESTRICTED` (on a `PUBLIC` door everyone can use
  it anyway, on a `PRIVATE` door Users cannot).
- Region owners do **not** become Admins of the door.
- Admins and the owner of the door can change it: door menu **Region access**, or
  `/dp set <door> region-access <true|false>`.
- Without WorldGuard the option can be saved but has no effect. The menu shows **WorldGuard is not active
  on this server.** and the command warns you.
- It only counts for online players.

Details on which regions count: [WorldGuard](../integrations/worldguard.md).

## How a use is decided

Every time a player tries to move a door (trigger, command, menu, group), Door+ runs these checks in this
order. The first check that decides stops the list.

1. **Locked.** A locked door does not move for anyone, staff with bypass permissions included:
   **`<door>` is locked.**
2. **Bypass.** The player has `doorplus.admin.bypass.use` → allowed.
3. **Role Admin or Owner** (from ownership, access entries) → allowed. No key needed.
4. **Trigger access mode.** If the use comes from a trigger whose access mode is not **Door privacy**
   (`INHERIT`), the trigger mode replaces the door privacy in the next steps. Redstone triggers have no
   access mode.
5. **Trigger mode Public** → allowed, without looking at the key.
6. **In the list?** Door+ decides whether the player is "in the list":

    | Mode used | In the list |
    |---|---|
    | Door privacy `PUBLIC` | everyone |
    | Door privacy `RESTRICTED` | players with role User (access entry or Region access) |
    | Door privacy `PRIVATE` | nobody |
    | Trigger mode **Restricted** | players matching one of the trigger's own entries (the door's access list does not count) |
    | Trigger mode **Owner and admins** | nobody |

7. **Key mode** of the door:

    | Key mode | Allowed if |
    |---|---|
    | **No key** | in the list |
    | **Key as alternative** | in the list, **or** holding the key |
    | **Key required** | in the list **and** holding the key |

8. Another plugin can still change the result through the Door+ API.

When the result is "denied" the player sees **You cannot use `<door>`.** Proximity zones never send this
message: players who are not allowed are simply not counted.

Uses without a player:

| Origin | Rule |
|---|---|
| Redstone trigger | Allowed only if the door has **Redstone** on and `triggers.redstone.enabled` is `true`. |
| Console and command blocks | Allowed (the lock still applies). |
| Auto-close | Allowed. |

## Admin bypass permissions

| Permission | Effect |
|---|---|
| `doorplus.admin.bypass.use` | Use any door, whatever the privacy, key and trigger access mode. Does not open locked doors. Also lets the player find any door with commands and menus. |
| `doorplus.admin.bypass.edit` | Act as at least **Admin** on any door. |
| `doorplus.admin.bypass.manage` | Act as **Owner** on any door, and manage other players' door groups. |
| `doorplus.admin.bypass.limits` | Ignore the per-player limits. |
| `doorplus.admin.bypass.protection` | Ignore protection plugins (WorldGuard and others) when creating doors and linking triggers. |

All are given to operators by default and are included in `doorplus.admin`. Full list:
[Permissions](../reference/permissions.md).

## Examples

### Shop door for a rank

Only players of the LuckPerms group `vip` may open the shop door.

1. Set the privacy to restricted: `/dp privacy shop restricted`.
2. Add the group as User: `/dp access add shop group vip`.
3. Optionally give your shop keeper the Admin role so they can lock it at night:
   `/dp access add shop player Alex admin`.

Players outside `vip` see **You cannot use shop.** when they press its button.

!!! tip
    Without LuckPerms, use a permission entry instead: `/dp access add shop permission myserver.shop`.

### Staff-only door

1. Keep the door `PRIVATE` (the default): `/dp privacy staffroom private`.
2. Give the staff group the Admin role: `/dp access add staffroom group staff admin`.

Only the owner and the staff can open it. If you want staff to use it without being able to change it, use
`RESTRICTED` with the role User instead.

### Public gate with a private control button

1. Make the gate public: `/dp privacy gate public`. Every player can now use its triggers.
2. Link the button in the guard house: `/dp trigger add gate block`, then click the button.
3. Set that trigger to owner and admins only:
   `/dp trigger set <trigger id> access owner-only`.

Every player can use the other triggers of the gate, but only you and the admins can use the guard house
button. See [Triggers](triggers.md#access-mode).

### Private house with a public exit button

1. Keep the door `PRIVATE`.
2. Link a button inside the house and set its access to public:
   `/dp trigger set <trigger id> access public`.

Anyone who is inside (for example a guest you let in) can leave, while the outside triggers still follow
the door privacy.
