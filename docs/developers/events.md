# Events

This page is for plugin developers. It lists every Bukkit event Door+ fires, when it fires, whether you can cancel
it, what data it carries, and a typical listener. For the dependency and how to get the API, see
[Developer API](api.md).

All events are in the package `io.github.cristiaaaaan.doorplus.api.event`. You listen to them like any Bukkit
event: implement `Listener`, annotate a method with `@EventHandler` and register the listener in `onEnable`.

!!! note "Thread"
    Every Door+ event is **synchronous** and fired on the **main thread**. You can use the Bukkit API directly in
    your listeners. Keep them fast: some events (access checks, movements) fire often.

## Overview

| Event | Cancellable | Fires |
|---|:-:|---|
| [`DoorCreateEvent`](#doorcreateevent) | yes | Before a new door is saved |
| [`DoorDeleteEvent`](#doordeleteevent) | yes | Before a door is deleted |
| [`DoorPreToggleEvent`](#doorpretoggleevent) | yes | After every check passed, just before a movement starts |
| [`DoorToggleStartEvent`](#doortogglestartevent) | no | When a movement starts |
| [`DoorToggleEndEvent`](#doortoggleendevent) | no | When a movement ends and the blocks are placed |
| [`DoorAccessCheckEvent`](#dooraccesscheckevent) | no (result can be changed) | Every time Door+ checks whether a player may use a door |
| [`DoorPrivacyChangeEvent`](#doorprivacychangeevent) | yes | Before the privacy of a door changes |
| [`DoorAccessChangeEvent`](#dooraccesschangeevent) | yes | Before an access entry is added, removed or its role changed |
| [`DoorOwnerChangeEvent`](#doorownerchangeevent) | yes | Before the ownership of a door is transferred |
| [`TriggerAddEvent`](#triggeraddevent) | yes | Before a trigger is linked |
| [`TriggerRemoveEvent`](#triggerremoveevent) | no | After a trigger has been removed |
| [`TriggerActivateEvent`](#triggeractivateevent) | yes | When a trigger source fires, before the door is operated |

## Class hierarchy

- `DoorEvent` (abstract): base of the events about an existing door. Getter: `getDoor()` returns the live `Door`.
    - `DoorToggleEvent` (abstract): common data of the movement events, see [below](#movement-data).
        - `DoorPreToggleEvent`, `DoorToggleStartEvent`, `DoorToggleEndEvent`
    - `DoorAccessCheckEvent`, `DoorDeleteEvent`, `DoorPrivacyChangeEvent`, `DoorAccessChangeEvent`,
      `DoorOwnerChangeEvent`
- `DoorCreateEvent`, `TriggerAddEvent`, `TriggerRemoveEvent`, `TriggerActivateEvent` extend Bukkit's `Event`
  directly (the door or trigger does not exist yet, or is not a door).

Listen to the concrete classes. The abstract classes have no handler list.

## Movement events

A movement goes through three events, in this order:

1. `DoorPreToggleEvent`: every check passed (door, world, pause, lock, access, target state, busy, cooldown,
   obstacles, protection hooks). Cancel it to refuse the movement: the requester gets the result `CANCELLED`.
2. `DoorToggleStartEvent`: the movement has started.
3. `DoorToggleEndEvent`: the movement is over and the blocks are at the destination.

For an instant movement (no animation) the start and end events fire one right after the other.

!!! note "Perpetual types and revolving doors"
    - Windmill, flag and clock: starting them (state 0 → 1) fires the three events at once, as soon as the door is
      running. Stopping them (1 → 0) fires the pre and start events immediately and the end event when the blocks
      are back in their rest position.
    - Revolving door: its only state is 0, so `getFromState()` and `getToState()` are both 0 for each rotation.
    - Platform: states are stop indices (0 = first stop).

### Movement data

`DoorPreToggleEvent`, `DoorToggleStartEvent` and `DoorToggleEndEvent` share these getters (from `DoorToggleEvent`):

| Getter | Type | Meaning |
|---|---|---|
| `getDoor()` | `Door` | The door |
| `getFromState()` | `int` | State before the movement |
| `getToState()` | `int` | State after the movement |
| `getCause()` | `ToggleCause` | `PLAYER_TRIGGER`, `REDSTONE`, `PROXIMITY`, `COMMAND`, `GUI`, `API` or `AUTO_CLOSE` |
| `getActor()` | `UUID`, nullable | Player responsible; `null` for redstone, console and API calls without actor |
| `getTrigger()` | `Trigger`, nullable | Trigger that caused the movement, if any |
| `isInstant()` | `boolean` | `true` if the movement happens without animation |

### DoorPreToggleEvent

- **Fires**: after every check passed, just before a movement starts.
- **Cancellable**: yes. Cancelling refuses the movement with `ToggleResult.CANCELLED`. Door+ sends no message to the
  player for this result, so tell them yourself if needed.
- **Fields**: [movement data](#movement-data).
- **Typical use**: block a door during an event, a heist or a minigame phase.

```java
@EventHandler
public void onPreToggle(DoorPreToggleEvent event) {
    if (event.getDoor().getName().startsWith("vault_") && !heistActive) {
        event.setCancelled(true);
        UUID actor = event.getActor();
        Player player = actor == null ? null : Bukkit.getPlayer(actor);
        if (player != null) player.sendMessage("The vault is sealed.");
    }
}
```

### DoorToggleStartEvent

- **Fires**: when a movement starts (the blocks have left their cells and the animation, if any, is running).
- **Cancellable**: no.
- **Fields**: [movement data](#movement-data).
- **Typical use**: play an effect, start a timer, log who operated a door.

```java
@EventHandler
public void onStart(DoorToggleStartEvent event) {
    getLogger().info(event.getDoor().getName() + " " + event.getFromState() + " -> " + event.getToState()
            + " by " + event.getCause());
}
```

### DoorToggleEndEvent

- **Fires**: when a movement ends and the blocks are placed at the destination.
- **Cancellable**: no.
- **Fields**: [movement data](#movement-data).
- **Typical use**: react when a door is fully open or closed (open a second door, spawn mobs, update a scoreboard).

```java
import net.kyori.adventure.text.Component;

@EventHandler
public void onEnd(DoorToggleEndEvent event) {
    Door door = event.getDoor();
    if (door.getName().equals("arena_gate") && door.isOpen()) {
        Bukkit.broadcast(Component.text("The arena is open!"));
    }
}
```

## DoorAccessCheckEvent

- **Fires**: at the end of every "may this player use this door" check, with the result Door+ computed (privacy,
  role, key, trigger access override, bypass permission). This happens for movement requests made by a player
  (triggers, commands, menus), for players inside a proximity zone, for the `can_use` placeholders and for
  `AccessManager.canUse`.
- **Cancellable**: no, but you can change the result.
- **Fields**:

| Getter / setter | Type | Meaning |
|---|---|---|
| `getDoor()` | `Door` | The door |
| `getPlayer()` | `Player` | The player being checked |
| `getTrigger()` | `Trigger`, nullable | Trigger being used; `null` for commands and menus |
| `isAllowed()` | `boolean` | Current result |
| `setAllowed(boolean)` | | Replace the result |

- **Typical use**: add your own access rule (a quest, a rank, a team) on top of Door+.

!!! warning
    This event fires often and also for checks that do not move the door (placeholders, proximity zones). Do not
    send messages or change state in it: only compute the answer.

```java
@EventHandler
public void onAccessCheck(DoorAccessCheckEvent event) {
    // Members of the red team may always use doors whose name starts with "red_"
    if (event.getDoor().getName().startsWith("red_") && teams.isRed(event.getPlayer())) {
        event.setAllowed(true);
    }
}
```

## DoorCreateEvent

- **Fires**: before a new door is saved, as the last validation step of the guided creation and of
  `DoorManager.create`. It does not fire when an existing door's region is redefined.
- **Cancellable**: yes. The creator sees **The creation was cancelled by another plugin.**; an API creation fails.
- **Fields**:

| Getter | Type | Meaning |
|---|---|---|
| `getDraft()` | `DoorSpec` | The door about to be created: type, name, owner, world, region, pivot, direction, ... |
| `getPlayer()` | `Player`, nullable | The creator; `null` for API creation |

- **Typical use**: limit door sizes or types per area, charge money for a door.

```java
@EventHandler
public void onCreate(DoorCreateEvent event) {
    DoorSpec draft = event.getDraft();
    Player player = event.getPlayer();
    if (player != null && draft.region().volume() > 100 && !player.hasPermission("myplugin.bigdoors")) {
        event.setCancelled(true);
        player.sendMessage("Doors bigger than 100 blocks need a rank.");
    }
}
```

## DoorDeleteEvent

- **Fires**: before a door is deleted (command, menu or `DoorManager.delete`).
- **Cancellable**: yes. An API deletion then fails with `IllegalStateException`.
- **Fields**: `getDoor()`; `getActor()` (`UUID`, nullable): who deletes the door, `null` for console and API.
- **Typical use**: protect important doors, clean up your own data linked to the door.

```java
@EventHandler
public void onDelete(DoorDeleteEvent event) {
    if (protectedDoorIds.contains(event.getDoor().getId())) event.setCancelled(true);
}
```

## DoorPrivacyChangeEvent

- **Fires**: before the privacy of a door changes (command, menu or `Door.setPrivacy`).
- **Cancellable**: yes. A cancelled `Door.setPrivacy` call simply does nothing.
- **Fields**: `getDoor()`, `getOldPrivacy()`, `getNewPrivacy()` (`Privacy`: `PRIVATE`, `RESTRICTED`, `PUBLIC`),
  `getActor()` (`UUID`, nullable).
- **Typical use**: forbid public doors in some worlds.

```java
@EventHandler
public void onPrivacy(DoorPrivacyChangeEvent event) {
    World world = Bukkit.getWorld(event.getDoor().getWorldId());
    if (event.getNewPrivacy() == Privacy.PUBLIC && world != null && world.getName().equals("prison")) {
        event.setCancelled(true);
    }
}
```

## DoorAccessChangeEvent

- **Fires**: before an access entry is added, removed or its role changed (command, menu, `AccessManager.grant` and
  `revoke`).
- **Cancellable**: yes. A cancelled API `grant` or `revoke` does nothing and does not throw.
- **Fields**:

| Getter | Type | Meaning |
|---|---|---|
| `getDoor()` | `Door` | The door |
| `getSubject()` | `AccessSubject` | Player, LuckPerms group or permission (`type()`, `value()`) |
| `getOldRole()` | `Role`, nullable | `null` when the entry is being added |
| `getNewRole()` | `Role`, nullable | `null` when the entry is being removed |
| `getActor()` | `UUID`, nullable | Who makes the change |

- **Typical use**: audit logs, keep an external permission system in sync.

```java
@EventHandler
public void onAccessChange(DoorAccessChangeEvent event) {
    String what = event.getOldRole() == null ? "added"
            : event.getNewRole() == null ? "removed" : "changed";
    getLogger().info(event.getSubject().type() + " " + event.getSubject().value() + " " + what
            + " on door #" + event.getDoor().getId());
}
```

## DoorOwnerChangeEvent

- **Fires**: before the ownership of a door is transferred (`/dp transfer`, the transfer button in the menu, and
  `/dp admin claim`).
- **Cancellable**: yes.
- **Fields**: `getDoor()`, `getOldOwner()` (`UUID`), `getNewOwner()` (`UUID`), `getActor()` (`UUID`, nullable).
- **Typical use**: forbid selling or giving doors, update your own records.

```java
@EventHandler
public void onOwnerChange(DoorOwnerChangeEvent event) {
    if (frozenPlayers.contains(event.getOldOwner())) event.setCancelled(true);
}
```

## TriggerAddEvent

- **Fires**: before a trigger is linked, after Door+'s own checks passed (permission, role, distance, limits,
  duplicates, protection hooks). It fires for triggers linked by command, menu, during the guided creation and
  through `TriggerManager.add`.
- **Cancellable**: yes. The player sees **Another plugin prevented linking this trigger.**; an API call fails.
- **Fields**:

| Getter | Type | Meaning |
|---|---|---|
| `getDraft()` | `TriggerSpec` | The trigger about to be linked, with defaults already applied (target, type, provider, world, position, action, access mode, cooldown, ...) |
| `getPlayer()` | `Player`, nullable | The player linking it; `null` for API and import |

- **Typical use**: forbid some trigger types in some places.

```java
@EventHandler
public void onTriggerAdd(TriggerAddEvent event) {
    if (event.getDraft().type() == TriggerType.PROXIMITY && event.getDraft().radius() > 8) {
        event.setCancelled(true);
        if (event.getPlayer() != null) event.getPlayer().sendMessage("Proximity zones are limited to 8 blocks here.");
    }
}
```

## TriggerRemoveEvent

- **Fires**: after a trigger has been removed.
- **Cancellable**: no.
- **Fields**: `getTrigger()`: the removed trigger (a detached view: its values no longer change);
  `getCause()` (`TriggerRemoveCause`):

| Cause | Meaning |
|---|---|
| `MANUAL` | Removed by a player from a menu or command |
| `API` | Removed through `TriggerManager.remove` |
| `SOURCE_REMOVED` | The source entity, custom block, furniture or NPC disappeared |
| `OBSOLETE` | The custom object at the source position now has a different id |
| `TARGET_DELETED` | The target door or group was deleted |

- **Typical use**: clean up data you attached to a trigger id.

```java
@EventHandler
public void onTriggerRemove(TriggerRemoveEvent event) {
    myTriggerData.remove(event.getTrigger().getId());
}
```

## TriggerActivateEvent

- **Fires**: when a trigger source fires (click, redstone power, entering a zone, ...), before the door or group is
  operated. It also fires for `TriggerManager.activate`. It does not fire for a disabled trigger, for a redstone
  trigger while redstone triggers are disabled in the config, or while the trigger's own cooldown is running.
- **Cancellable**: yes. Nothing is operated.
- **Fields**:

| Getter | Type | Meaning |
|---|---|---|
| `getTrigger()` | `Trigger` | The trigger |
| `getActor()` | `Player`, nullable | Player that fired the source; `null` for redstone, for the release of a `FOLLOW` source, and for `TriggerManager.activate(trigger, null)` |
| `isActive()` | `boolean` | `true` when the source is activated (powered, entered); `false` when it is deactivated, which only happens for `FOLLOW` triggers |

- **Typical use**: require an item, money or a cooldown of your own to use a trigger.

```java
@EventHandler
public void onTriggerActivate(TriggerActivateEvent event) {
    Player player = event.getActor();
    if (player != null && event.getTrigger().getId() == 17 && !economy.has(player, 10)) {
        event.setCancelled(true);
        player.sendMessage("Opening this door costs 10 coins.");
    }
}
```

## Registering the listener

```java
public final class MyPlugin extends JavaPlugin implements Listener {
    @Override
    public void onEnable() {
        getServer().getPluginManager().registerEvents(this, this);
    }

    @EventHandler
    public void onEnd(DoorToggleEndEvent event) {
        // ...
    }
}
```

With `softdepend: [DoorPlus]`, register a listener that references Door+ classes only when Door+ is present
(`getServer().getPluginManager().getPlugin("DoorPlus") != null`), otherwise Bukkit cannot load the listener class.
