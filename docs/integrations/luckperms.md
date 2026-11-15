# LuckPerms

This page explains what Door+ does with LuckPerms: giving a whole permission group access to a door. It is for
server administrators and door owners.

## What LuckPerms adds

With LuckPerms installed you can add **group** access entries to a door: every member of the LuckPerms group gets
the role you choose (**User** or **Admin**). Without LuckPerms you can still add player entries and permission
entries, but not group entries.

| | With LuckPerms | Without LuckPerms |
|---|---|---|
| Add a group entry | Yes. Menus and tab completion list the groups LuckPerms has loaded. | No: the command answers **Group entries require LuckPerms.** and the menu button shows **Requires LuckPerms.** |
| Group entries already saved | Checked normally | Still checked through the `group.<name>` permission (usually nobody has it) |

The same group entries work in the access list of a trigger (see [Triggers](../access/triggers.md)).

## Giving a group access to a door

You need the **Owner** role on the door (or the bypass permissions of an operator) and the permission
`doorplus.access.group`, which every player has by default through `doorplus.user`.

=== "Menu"

    1. Open the door menu and click **Access list**.
    2. Click **Add LuckPerms group**. The menu **Choose a LuckPerms group** opens.
    3. Click the group.
    4. Choose the role (**User** or **Admin**).

=== "Command"

    ```
    /dp access add <door> group <group> [user|admin]
    ```

    Example: `/dp access add castle_gate group guards admin`. Without a role the entry gets **User**.
    To remove it: `/dp access remove castle_gate group guards`.

Group names are stored in lower case and can contain letters, digits, `_`, `.` and `-` (at most 64 characters).

Remember that only **Restricted** doors let access entries with the **User** role in. **Admin** entries can use
the door with any privacy. See [Privacy and roles](../access/privacy-and-roles.md).

## How membership is checked

For a player who is online, Door+ checks the permission `group.<name>`, which LuckPerms gives to every member of a
group, including members through inheritance. There is no cache on the Door+ side: when you add a player to a group
or remove them, the change applies to doors immediately.

If you delete a group in LuckPerms, the entry stays on the door and the access list shows it in red with
**This LuckPerms group no longer exists.** Remove it with shift + right click in the menu or with
`/dp access remove`.

!!! note
    Checks for players who are **offline** (only used by other plugins through the Door+ API) ask LuckPerms for the
    player's groups and permissions. Without LuckPerms, offline checks only see player entries.

## Limits per group

Door+ limits can be raised with permission nodes (`doorplus.limit.doors.<n>`, `doorplus.limit.size.<n>`,
`doorplus.limit.triggers.<n>`). With LuckPerms you give them to a group like any other permission:

```
/lp group vip permission set doorplus.limit.doors.25 true
```

The highest number a player has wins. See [Limits](../reference/limits.md) and
[Permissions](../reference/permissions.md).
