# WorldGuard

This page explains how Door+ respects WorldGuard regions, the two Door+ flags, and region access for doors. It is
for server administrators who protect their worlds with WorldGuard.

With WorldGuard installed, Door+ adds five things:

| Feature | Applies to |
|---|---|
| [Build check on creation](#build-check-when-creating-a-door) | Creating and redefining doors |
| [Flag `doorplus-create`](#flag-doorplus-create) | Creating and redefining doors |
| [Flag `doorplus-toggle`](#flag-doorplus-toggle) | Every door movement |
| [Build check on trigger linking](#build-check-when-linking-a-trigger) | Linking a trigger source |
| [Region access](#region-access) | Who can use a door |

## Build check when creating a door

To create a door, or to redefine its region, a player must be allowed to build in every WorldGuard region that
touches:

- the door itself (its closed position), and
- the space the door occupies when it is open.

If one region refuses, the creation stops with **You cannot create a door here (denied by WorldGuard).**

This uses WorldGuard's normal build rules: region members and owners can build, the `build` flag applies, and so do
global region settings.

## Flag `doorplus-create`

A state flag, default **allow**. Set it to `deny` to stop everyone, members and owners included, from creating doors
in a region:

```
/rg flag castle doorplus-create deny
```

The check covers the same space as the build check above. To remove the flag again:

```
/rg flag castle doorplus-create
```

## Flag `doorplus-toggle`

A state flag, default **allow**. Set it to `deny` to stop doors from moving **into** a region:

```
/rg flag spawn doorplus-toggle deny
```

Door+ checks the position the door is moving to. A door whose open position touches the region cannot open; a door
whose closed position touches the region cannot close. Windmills, flags and clocks cannot be started if their blocks
touch the region.

The flag applies to every movement, whoever or whatever starts it (players, triggers, redstone, commands, the
console, auto-close), and there is no bypass. A refused movement shows **&lt;door&gt; cannot move here: denied by
WorldGuard.** to the player who started it.

## Build check when linking a trigger

To link a trigger, a player must be allowed to build at the position of the source (the button, block, entity,
furniture or NPC). Otherwise the link is refused with **The protection WorldGuard does not allow you to link this
source.** See [Triggers](../access/triggers.md).

## Bypass

| Who | Skips |
|---|---|
| Players with WorldGuard's region bypass in that world | The build checks and `doorplus-create` |
| Players with `doorplus.admin.bypass.protection` (operators by default) | The build checks and `doorplus-create`, for every protection plugin |

Nothing bypasses `doorplus-toggle`.

## Region access

Region access lets the members of a WorldGuard region use a door without adding them one by one.

When region access is on for a door, every player who is a **member or owner** of a WorldGuard region containing the
centre of the door (closed position) gets the **User** role on that door, unless they already have a higher role.

- The door must be **Restricted**: like any User entry, region access does nothing on **Private** doors, and
  **Public** doors are open to everyone anyway.
- Members of parent regions and WorldGuard groups count. The `__global__` region does not.
- Region owners get **User**, not **Admin**.
- Only online players are checked.

To switch it on (you need the **Admin** role on the door):

=== "Menu"

    1. Open the door menu.
    2. Click **Region access**. The lore shows **WorldGuard region members: on**.

=== "Command"

    ```
    /dp set <door> region-access true
    ```

    Use `false` to switch it off.

New doors start with the value of `defaults.region-access` (default `false`).

If WorldGuard is not active, you can still switch the option on, but it has no effect: the command warns
**WorldGuard is not active: region access has no effect until it is installed.** and the menu shows **WorldGuard is
not active on this server.**

!!! warning
    Region access works only on **Restricted** doors. When the door is **Private**, the region access button shows
    **The door is PRIVATE: set it to RESTRICTED for region access to work.**, and switching region access on (or
    making such a door Private) by command shows **`<door>` is PRIVATE: region members can open it only when it is
    RESTRICTED (/door privacy `<door>` restricted).** See [Privacy and roles](../access/privacy-and-roles.md).

## Troubleshooting

At startup the console shows which features are on, for example:

```
WorldGuard 7.0.17: active (protection hook, region access, flags doorplus-create, doorplus-toggle)
```

- **`custom flags unavailable`** at the end of the line: the flags could not be registered. Door+ registers them
  while the server starts; if Door+ is loaded later by a plugin manager, WorldGuard no longer accepts new flags.
  Restart the server. The build checks and region access still work without the flags.
- **`/rg flag` says the flag does not exist**: same cause, restart the server.
- Setting `integrations.worldguard: false` turns off all the checks above and region access. The two flags are still
  registered when WorldGuard is installed, but have no effect.
