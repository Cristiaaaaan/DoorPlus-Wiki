# NPCs

This page explains how to make a Citizens or FancyNpcs NPC open a door when a player right-clicks it. It is for
server owners and door owners on servers with one of these plugins.

With Citizens or FancyNpcs installed, an NPC can be a trigger of type **NPC**: a gatekeeper that opens the castle
gate, a lift operator that sends a platform to a stop, and so on. No configuration is needed. Each plugin can be
switched off with `integrations.citizens` or `integrations.fancynpcs` in `config.yml` (see
[Integrations](index.md)).

## Linking an NPC

You need the **Admin** role on the door and the permissions `doorplus.trigger.entity` and `doorplus.trigger.npc`
(every player has them by default through `doorplus.user`).

=== "Menu"

    1. Open the door menu and click **Triggers**.
    2. Click **Add trigger**, then **Entity**.
    3. Right-click the NPC.

=== "Command"

    1. Run `/dp trigger add <door> entity [action]`, e.g. `/dp trigger add castle_gate entity toggle`.
    2. Right-click the NPC.

You see **Linked NPC trigger #&lt;id&gt; to &lt;door&gt;.** You have `triggers.selection-timeout-seconds` seconds
(default 30) to click; type `/dp cancel` to stop.

The usual trigger rules apply when linking (see [Triggers](../access/triggers.md)):

- The NPC must be in the same world as the door.
- It must be within `limits.max-trigger-distance` blocks of the door (default 64), measured where the NPC stands when
  you link it.
- With WorldGuard, you must be allowed to build where the NPC stands (see [WorldGuard](worldguard.md)).

If Door+ answers **This object is not recognised yet. Try again in a moment.**, neither Citizens nor FancyNpcs
reported the clicked NPC as one of theirs. Try again; if the message repeats, check the NPC in its own plugin.

## How NPC triggers behave

- The trigger fires when a player **right-clicks** the NPC. Left clicks do nothing.
- Available actions: **Toggle**, **Open**, **Close**, and **Go to stop** for platforms. **Follow state** is not
  available.
- The player must be allowed to use the door, as for any trigger. To let everyone use the door through the NPC
  while the door itself stays private, change the access of the trigger (see [Triggers](../access/triggers.md)).
- By default the NPC's own click actions (Citizens commands, FancyNpcs actions) still run. To stop them when the
  click is used by Door+, run `/dp trigger set <trigger id> cancel-vanilla true`.
- The trigger is tied to the NPC itself, not to a position: you can move or teleport the NPC and the trigger keeps
  working.

Door+ identifies NPCs by an id that survives restarts: the NPC's UUID for Citizens, the NPC id from the save file
for FancyNpcs.

To see which triggers an NPC has, run `/dp trigger inspect` and right-click the NPC.

## When the NPC is deleted

| What happens | Result |
|---|---|
| The NPC is deleted in Citizens or FancyNpcs | Its triggers are removed. If the door owner is online, they get **The source of trigger #&lt;id&gt; (NPC) of &lt;door&gt; at x y z disappeared: the trigger was removed.** The coordinates are where the NPC was when it was linked. |
| The server stops, or Citizens unloads its NPCs while shutting down | Nothing: the triggers stay. |
| The NPC plugin is uninstalled | The triggers stay saved but do nothing. They work again if you reinstall the plugin and the NPC still exists. |

A trigger can always be removed by hand from the trigger menu or with `/dp trigger remove <trigger id>`.
