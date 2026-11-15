# Triggers

A trigger links a **source** in the world (a button, any block, a redstone line, an entity, a custom block,
an NPC or a zone) to a **target**: one door or one [door group](groups.md). This page lists every trigger
type, how to link and edit triggers, and exactly what each setting does. It is for door owners, door
admins and staff.

## Basics

- A door can have many triggers, and one source can control many targets (one trigger per target).
- Each trigger has its own action, access mode, cooldown and other settings.
- Managing the triggers of a door requires the role **Admin** or **Owner** on it (see
  [Privacy and roles](privacy-and-roles.md)). Triggers of a group are managed by the group owner.
- Linking a trigger of a type requires the permission `doorplus.trigger.<type>`.
- Every trigger has a number (id), shown as `#12`. Commands accept `12` or `#12`.

## Trigger types

| Type | Menu name | Source | Activated by | Permission |
|---|---|---|---|---|
| Block | **Block** | Any placed vanilla block: button, lever, pressure plate, tripwire, but also a chest or a bookshelf | Right click with the main hand; stepping on a pressure plate or tripwire | `doorplus.trigger.block` |
| Redstone | **Redstone** | Any block position that receives redstone power | Redstone power turning on (and off, with **Follow state**) | `doorplus.trigger.redstone` |
| Entity | **Entity** | A vanilla entity of a type in `triggers.entity.allowed-types` | Right click on the entity | `doorplus.trigger.entity` |
| Custom block | **Custom block** | A placed custom block of ItemsAdder, Nexo, Oraxen or CraftEngine | Clicking it | `doorplus.trigger.custom_block` |
| Furniture | **Furniture** | A placed furniture of ItemsAdder, Nexo, Oraxen or CraftEngine | Clicking it | `doorplus.trigger.furniture` |
| NPC | **NPC** | A Citizens or FancyNpcs NPC | Clicking it | `doorplus.trigger.npc` |
| Proximity | **Proximity** | A spherical zone (centre and radius) | An allowed player entering the zone | `doorplus.trigger.proximity` |

`doorplus.trigger.*` gives all of them and is included in `doorplus.user`, which every player has by
default. Custom blocks, furniture and NPCs need the matching plugin: see
[Custom blocks and furniture](../integrations/custom-blocks.md) and [NPCs](../integrations/npcs.md).

### Block

- Right click with the main hand activates the trigger. Pressure plates and tripwires activate when
  pressed (once per press, not every tick while you stand on them).
- The click is ignored if you are sneaking while holding an item (so you can place blocks against the
  source), if you hold the creation stick, or if you are creating a door or selecting a trigger.
- **Follow state** is available only for blocks that can be powered (buttons, levers, pressure plates,
  tripwires, and other blocks with a powered state). The trigger is active while the block is powered.
- With **Follow state**, only a power change started by a player counts: a pressure plate pressed by a
  mob, an armor stand or a dropped item does not open the door. When the player steps off, the door
  follows the plate back.
- The trigger is bound to the position. It stays if the block is broken or replaced.

### Redstone

- "Powered" means the block at the trigger position is powered directly or indirectly. Door+ checks the
  position and its six neighbours one tick after each redstone change, block placement or block break.
- **Toggle**, **Open** and **Close** run when the power turns on. **Follow state** opens when the power
  turns on and closes when it turns off.
- It works only if the door has **Redstone** on (door menu **Redstone**, or
  `/dp set <door> redstone true`) and `triggers.redstone.enabled` is `true` in the config. Linking a
  redstone trigger to a door with **Redstone** off shows a warning:
  **Redstone is disabled on `<door>`: enable "Allow redstone" in its settings for this trigger to work.**
- When a chunk loads, Door+ reads the current power without moving any door.
- Redstone triggers have no player, so they have no access mode and no own entries.
- The trigger is bound to the position. It stays if the block changes.

!!! warning
    Whoever can reach the circuit can open a door with **Redstone** on. Keep the wiring protected.

### Entity

- Allowed entity types come from `triggers.entity.allowed-types` (default `ITEM_FRAME`,
  `GLOW_ITEM_FRAME`, `ARMOR_STAND`, `INTERACTION`, `PAINTING`). Players can never be used. Another type
  gives **This kind of entity cannot be used as a trigger.**
- The trigger is removed when the entity dies, breaks or is removed from the world (not when its chunk
  unloads).

!!! tip
    Display entities have no hitbox. To use one as a button, place an `INTERACTION` entity on it and
    link that.

### Custom block and furniture

- Door+ listens to the click events of the custom content plugin.
- In selection mode, clicking a block that a custom content plugin recognises links a **Custom block**;
  clicking an entity it recognises links a **Furniture**.
- If the plugin recreates a furniture with a new entity, Door+ finds it again by its id and position.
- If the object at that position has a different id when clicked, the trigger is removed. It is also
  removed when the object is broken.

### NPC

- Works with Citizens and FancyNpcs. The trigger is bound to the NPC id and is removed when the NPC is
  deleted.

### Proximity

- The zone is a sphere. The radius goes from `1` to `limits.max-proximity-radius` (default `16`).
- The centre is either the centre of the door (for a group: of its doors) or your position when you create
  the trigger.
- Every `triggers.proximity.check-interval-ticks` ticks (default `10`) Door+ looks for players inside the
  zones in loaded chunks.
- Only players who are allowed to use the door count (full check, see
  [How a use is decided](privacy-and-roles.md#how-a-use-is-decided)). Spectators never count. With
  `triggers.proximity.ignore-vanished: true` (default) vanished players do not count either. Players who
  are not allowed receive no message.
- **Follow state** (default for proximity): the door opens when the first allowed player enters and
  closes **Exit delay** ticks after the last one leaves.
- **Toggle**, **Open** and **Close**: run when the first allowed player enters an empty zone.

## Actions

| Action | Menu name | Effect | Available for |
|---|---|---|---|
| `TOGGLE` | **Toggle** | Opens a closed door, closes an open door. | Every source |
| `OPEN` | **Open** | Moves the door to open. Nothing happens if it is already open. | Every source |
| `CLOSE` | **Close** | Moves the door to closed. Nothing happens if it is already closed. | Every source |
| `FOLLOW` | **Follow state** | Source active = open, source inactive = closed. | Redstone, Proximity, Block on a block that can be powered |
| `GOTO` | **Go to stop** | Moves a platform to the stop given by **arg**. | Targets with at least one platform |

Default action when you link a trigger without choosing one: **Follow state** for Redstone and Proximity,
**Toggle** for every other type.

How each door type reacts to the actions is described in [Movement](../doors/movement.md).

### Follow state in detail

- A door with several **Follow state** sources (on the door itself or on its groups) stays open while
  **at least one** of them is active. It closes when the last one turns off.
- While a source is active, the door's auto-close does not run. Auto-close starts again after the last
  source turns off.
- If the door is still moving when a source changes, Door+ checks again at the end of the movement and
  moves the door to match the sources.
- Redstone sources only count for doors with **Redstone** on.
- With pressure plates, entities other than players are ignored (see [Block](#block)).

## Link a trigger

When you start linking, Door+ enters **selection mode**:

- Your inventory closes and chat tells you what to click and how many seconds you have
  (`triggers.selection-timeout-seconds`, default `30`).
- Left or right click the source. The click does not do anything else (a chest does not open, a lever does
  not flip).
- After a successful link the source shows green particles for 5 seconds and chat offers
  **[Add another]** and **[Done]**. **[Add another]** lets you click one more source for the same target.
- If linking fails, chat explains why and you can click another source.
- Type `/dp cancel` to stop. If you do nothing until the time runs out: **Selection timed out.** The timer
  restarts after each link.

=== "Menu"

    1. Open the door menu with `/dp menu <door>` (or the group menu, see [Groups](groups.md)).
    2. Click **Triggers**. The menu **Triggers — `<door>`** lists the current triggers.
    3. Click **Add trigger**. The menu **New trigger — `<door>`** opens.
    4. Choose the source:
        - **Block**: a block, button, lever, plate or custom block. Requires `doorplus.trigger.block` or
          `doorplus.trigger.custom_block`.
        - **Redstone**: a block that receives redstone power.
        - **Entity**: an entity, a furniture or an NPC. Requires `doorplus.trigger.entity`,
          `doorplus.trigger.furniture` or `doorplus.trigger.npc`.
        - **Proximity**: a zone, see [Link a proximity zone](#link-a-proximity-zone).
    5. Click the source in the world. Chat confirms:
       **Linked `<type>` trigger #`<id>` to `<door>`.**
    6. Click **[Done]**. The trigger list opens again.

=== "Command"

    ```
    /dp trigger add <door|group:group> <block|redstone|entity|proximity> [action] [radius]
    ```

    - `block` also accepts custom blocks, `entity` also accepts furniture and NPCs. The command needs the
      permission of the word you type (`doorplus.trigger.block`, ...); the source you click then needs the
      permission of its real type (for example `doorplus.trigger.custom_block`).
    - `[action]` is `toggle`, `open`, `close`, `follow` or `goto`. Without it the default action is used.
    - `[radius]` is only for `proximity`.

    Examples:

    ```
    /dp trigger add castle block
    /dp trigger add castle redstone follow
    /dp trigger add group:gates block open
    ```

=== "During creation"

    The guided creation has a trigger step: **Add triggers to open the door (optional).**

    1. Click **[Add trigger]**.
    2. Click **[Block]**, **[Redstone]**, **[Entity]** or **[Proximity]**.
    3. Click the source. Chat confirms: **Captured `<type>` at `<x>, <y>, <z>`. It will be linked when the
       door is created.** For **[Proximity]**, type the radius in chat, then click **[Door centre]** or
       **[My position]**.
    4. Repeat for more sources, then click **[Done]**.

    When the door is saved, the sources are linked one by one with the same checks as below, and chat
    reports **Triggers linked: `<count>`/`<total>`.** Sources linked this way use **Follow state** for
    Redstone and Proximity and **Toggle** for the others. See [Creating doors](../doors/creating.md).

### Link a proximity zone

1. Start linking a **Proximity** trigger from the menu, or run
   `/dp trigger add <door> proximity [action] [radius]`.
2. If you did not give a valid radius, chat asks:
   **Type the radius of the zone in chat, from 1 to `<max>` blocks.** Type a whole number.
3. Chat shows **Radius `<radius>`. Choose the centre of the zone:** with two buttons.
4. Click **[Centre of the door]** or **[My position]**. The trigger is linked.

### Checks when linking

Door+ checks these conditions in this order. The first one that fails stops the link with its message.

| Check | Message when it fails |
|---|---|
| You have `doorplus.trigger.<type>` for the source type. | **You do not have permission to link `<type>` triggers.** |
| You are Admin or Owner of the door. For a group target: of every door of the group (an empty group: you are its owner). | **You need at least the Admin role on `<door>` to manage its triggers.** |
| The source is in the same world as the door. | **The source must be in the same world as `<door>`.** |
| The source is at most `limits.max-trigger-distance` blocks (default `64`, `0` = unlimited) from the door. | **The source is too far from `<door>` ...** |
| The target has fewer triggers than your limit. | **`<door>` already has the maximum number of triggers (`<limit>`).** |
| The source is not a block of a door (it would move with it). | **The source cannot be a block of a door: it would move with it.** |
| The same source is not already linked to the same target with the same type. | **This source is already linked to `<door>` with the same type.** |
| An entity is of an allowed type. | **This kind of entity cannot be used as a trigger.** |
| The action is available for this source and target. | **The action `<action>` is not available for this source and target.** |
| A protection plugin (for example WorldGuard) lets you build there. Skipped with `doorplus.admin.bypass.protection`. | **The protection `<hook>` does not allow you to link this source.** |

The trigger limit is `limits.max-triggers-per-door` (default `16`) per door or group. It can be raised per
player with `doorplus.limit.triggers.<n>` and ignored with `doorplus.admin.bypass.limits`. See
[Limits](../reference/limits.md).

## Trigger settings

| Setting | Values | Default | Applies to |
|---|---|---|---|
| **Action** (`action`) | See [Actions](#actions) | **Follow state** for Redstone and Proximity, **Toggle** otherwise | All |
| **Stop** (`arg`) | Stop number, from `0` | `0` | Only with **Go to stop** |
| **Access** (`access`) | `inherit`, `public`, `restricted`, `owner-only` | `inherit` (**Door privacy**) | All except Redstone |
| **Cooldown** (`cooldown`) | Ticks (20 ticks = 1 second), `0` = none | `triggers.default-cooldown-ticks` (default `10`) | All |
| **Cancel vanilla interaction** (`cancel-vanilla`) | `true`, `false` | See below | Block, Entity, Custom block, Furniture, NPC |
| **Enabled** (`enabled`) | `true`, `false` | `true` | All |
| **Radius** (`radius`) | `1` to `limits.max-proximity-radius` | Chosen when linking | Proximity |
| **Exit delay** (`exit-delay`) | Ticks, `0` = immediately | `20` | Proximity with **Follow state** |

- **Stop**: for a door target, up to the last stop of the platform. For a group target, up to the highest
  stop of its platforms (menu) or `limits.max-platform-stops` − 1 (command).
- **Cooldown**: after a trigger fires, it ignores new activations for that many ticks. A **Follow state**
  change that arrives during the cooldown is applied when the cooldown ends. The door also has its own
  cooldown, `movement.toggle-cooldown-ticks`.
- **Cancel vanilla interaction**: when on, the normal effect of the click is cancelled (a linked chest does
  not open, an item frame does not rotate its item). Default: off for buttons, levers, pressure plates and
  tripwires (they still click and animate), on for every other block, entities, custom blocks and
  furniture, off for NPCs.
- **Enabled**: a disabled trigger keeps its settings but does nothing.

### Access mode

The access mode replaces the door privacy for uses through this trigger only.

| Mode | Menu name | Who can use this trigger |
|---|---|---|
| `inherit` | **Door privacy** | Follows the door privacy and access list. |
| `public` | **Public** | Everyone, whatever the privacy and the key. |
| `restricted` | **Restricted** | The owner, the admins and the trigger's own entries. Door Users who are not in the own entries are refused. |
| `owner-only` | **Owner and admins** | Only the owner and the admins. |

The door key still applies to **Door privacy**, **Restricted** and **Owner and admins** (for example, with
**Key as alternative** a key holder can use an **Owner and admins** trigger). The lock always applies. Full
order of checks: [How a use is decided](privacy-and-roles.md#how-a-use-is-decided).

For a group target, the access mode applies to each door of the group.

### Own entries (Restricted triggers)

A trigger with access **Restricted** has its own list of players, LuckPerms groups and permission nodes,
without roles. At most `limits.max-access-entries` entries (default `64`). Empty list: only the owner and
the admins can use the trigger.

=== "Menu"

    1. In the trigger menu, set **Access** to **Restricted** (left click: next, right click: previous).
    2. Right click **Access**. The menu **Own entries — trigger #`<id>`** opens.
    3. Click **Add player**, **Add LuckPerms group** or **Add permission**, as in the
       [access list](privacy-and-roles.md#add-an-entry). There is no role to choose.
    4. To remove an entry, ++shift+right-button++ it.

=== "Command"

    ```
    /dp trigger access <trigger id> <add|remove> <player|group|permission> <subject>
    ```

    Example: `/dp trigger access 12 add group vip`. Adding group entries requires LuckPerms and
    `doorplus.access.group`, adding permission entries requires `doorplus.access.permission`.

### Edit settings

=== "Menu"

    1. Open **Triggers** from the door (or group) menu.
    2. Click a trigger. The menu **Trigger #`<id>` — `<type>`** opens.
    3. Use the buttons:
        - **Action**: left click next, right click previous (only the actions available for this trigger).
        - **Stop**: left +1, right −1 (shown only with **Go to stop**).
        - **Access**: left next, right previous; right click opens the own entries when the mode is
          **Restricted**.
        - **Cooldown**, **Exit delay**: left +1, right −1, with shift ×10.
        - **Radius**: left +1, right −1.
        - **Cancel vanilla interaction**, **Enabled**: click to toggle.
        - **Highlight**: shows particles on the source for 10 seconds (a ring for proximity zones).
        - **Remove**: removes the trigger after confirmation.

=== "Command"

    ```
    /dp trigger set <trigger id> <action|arg|access|cooldown|cancel-vanilla|enabled|radius|exit-delay> <value>
    ```

    Examples:

    ```
    /dp trigger set 12 action open
    /dp trigger set 12 access restricted
    /dp trigger set 12 cooldown 40
    /dp trigger set 15 radius 6
    /dp trigger set 15 exit-delay 60
    ```

    `cooldown` and `exit-delay` accept `0` to `72000`. An option that does not apply to the trigger gives
    **The option `<option>` does not apply to this trigger.**

### List triggers

```
/dp trigger list <door|group:group>
```

Shows id, type, provider, position, action, stop, access mode and whether each trigger is enabled.

## Inspect a source

Inspect mode tells you which triggers are linked to a block or entity.

1. Run `/dp trigger inspect`, or click **Inspect** in a trigger list.
2. Click a block, entity, furniture or NPC within the selection time.
3. Chat lists each trigger with id, type, target, action and state. It shows only triggers of doors on
   which you have at least the role User; otherwise:
   **No triggers you can see are linked to this source.**

Inspect mode ends after one click.

## Remove a trigger

=== "Menu"

    Open the trigger, click **Remove**, then confirm.

=== "Command"

    ```
    /dp trigger remove <trigger id>
    ```

Triggers are also removed automatically:

| Source | Removed when |
|---|---|
| Block, Redstone | Never by itself: the trigger is bound to the position. |
| Custom block, Furniture | The object is broken, or it has a different id when clicked. |
| Entity | The entity dies, breaks or is removed (not on chunk unload). |
| NPC | The NPC is deleted. |
| All | The target door or group is deleted. |

When a source disappears, the door owner (if online) is told where the trigger was.

!!! tip
    With **Protect blocks** on (door menu **Protect blocks**), players without the role Admin cannot
    break or change the trigger sources of the door (blocks and entities). See
    [Editing doors](../doors/editing.md).

## Triggers on a group

A trigger can target a [door group](groups.md) instead of a door:

- In the menu, open the group and click **Triggers**. In commands, write the target as `group:<group>`,
  for example `/dp trigger add group:gates block`.
- Only the group owner (or `doorplus.admin.bypass.manage`) can list and edit them. To link one you also
  need the role Admin on every door of the group.
- The action runs on every door of the group, each with its own access check. **Toggle** keeps the doors
  in sync: if at least one door is closed, all open; otherwise all close.
- The distance and world checks use every door of the group.
- Deleting the group deletes its triggers.

## Limits

| Config key | Default | Meaning |
|---|---|---|
| `limits.max-triggers-per-door` | `16` | Triggers per door or per group. Per-player override `doorplus.limit.triggers.<n>`. |
| `limits.max-trigger-distance` | `64` | Maximum distance between source and door, in blocks. `0` = unlimited. |
| `limits.max-proximity-radius` | `16` | Maximum radius of a proximity zone. |
| `limits.max-access-entries` | `64` | Maximum own entries of a trigger (and access entries of a door). |
| `triggers.default-cooldown-ticks` | `10` | Cooldown of new triggers. |
| `triggers.selection-timeout-seconds` | `30` | Time to click a source in selection and inspect mode. |
| `triggers.redstone.enabled` | `true` | `false` turns off every redstone trigger on the server. |
| `triggers.entity.allowed-types` | `ITEM_FRAME`, `GLOW_ITEM_FRAME`, `ARMOR_STAND`, `INTERACTION`, `PAINTING` | Entity types that can be triggers. |
| `triggers.proximity.check-interval-ticks` | `10` | How often proximity zones are checked. |
| `triggers.proximity.ignore-vanished` | `true` | Vanished players do not count in proximity zones. |

All configuration keys: [Configuration](../reference/configuration.md).
