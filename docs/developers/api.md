# Developer API

This page is for plugin developers who want to read or control Door+ doors from their own plugin. It covers the
dependency, how to get the API, the threading rules and the main services, with examples that compile against
`DoorPlus-API` 1.0.0.

The API is a separate artifact, released under the MIT licence. It contains only interfaces, enums, records and
events in the package `io.github.cristiaaaaan.doorplus.api`. The Door+ plugin jar ships the same classes at the same
version, so your plugin compiles against the artifact and finds the implementation at runtime.

| Requirement | Value |
|---|---|
| Java | 21 |
| Server | Paper 1.21.4 or newer |
| Coordinates | `com.github.Cristiaaaaan:DoorPlus-API:1.0.0` (JitPack) |
| Versioning | Semantic: the Git tag is the version; an incompatible change raises the major version |

## Adding the dependency

Add JitPack as a repository and Door+ as a **compile-only** (Maven: `provided`) dependency.

=== "Gradle (Kotlin)"

    ```kotlin
    repositories {
        maven("https://jitpack.io")
    }
    dependencies {
        compileOnly("com.github.Cristiaaaaan:DoorPlus-API:1.0.0")
    }
    ```

=== "Gradle (Groovy)"

    ```groovy
    repositories { maven { url 'https://jitpack.io' } }
    dependencies { compileOnly 'com.github.Cristiaaaaan:DoorPlus-API:1.0.0' }
    ```

=== "Maven"

    ```xml
    <repositories>
      <repository>
        <id>jitpack.io</id>
        <url>https://jitpack.io</url>
      </repository>
    </repositories>
    <dependencies>
      <dependency>
        <groupId>com.github.Cristiaaaaan</groupId>
        <artifactId>DoorPlus-API</artifactId>
        <version>1.0.0</version>
        <scope>provided</scope>
      </dependency>
    </dependencies>
    ```

!!! warning "Never shade the API"
    Do not shade or relocate the API into your jar. The classes must come from the Door+ jar at runtime, otherwise
    your plugin and Door+ see two different copies and nothing works.

## plugin.yml

Door+ is registered on the server as `DoorPlus`. Declare it so that your plugin loads after it.

```yaml
depend: [DoorPlus]      # your plugin requires Door+
# or
softdepend: [DoorPlus]  # Door+ is optional: check DoorPlusProvider.isAvailable() before using the API
```

## Getting the API

There are two equivalent ways. Both give the same `DoorPlusApi` object.

```java
import io.github.cristiaaaaan.doorplus.api.DoorPlusApi;
import io.github.cristiaaaaan.doorplus.api.DoorPlusProvider;
import org.bukkit.Bukkit;

// 1. Static provider: throws IllegalStateException("Door+ is not enabled") when Door+ is not running
DoorPlusApi api = DoorPlusProvider.get();

// 2. Bukkit ServicesManager: null when Door+ is not running
DoorPlusApi fromServices = Bukkit.getServicesManager().load(DoorPlusApi.class);
```

| Method | Returns |
|---|---|
| `DoorPlusProvider.get()` | The running API; throws `IllegalStateException` if Door+ is not enabled |
| `DoorPlusProvider.isAvailable()` | `true` while Door+ is enabled |
| `DoorPlusProvider.version()` | Version of the API classes, read from the jar (`unknown` if missing) |
| `api.apiVersion()` | Version of the API implemented by the running plugin |

With `softdepend`, check first:

```java
if (DoorPlusProvider.isAvailable()) {
    DoorPlusApi api = DoorPlusProvider.get();
    // ...
}
```

!!! note "When Door+ is disabled"
    Door+ publishes the API at the end of its start-up and removes it when it disables (for example on shutdown,
    after a start-up error, or when the purchase check fails, see
    [Licence, verification and privacy](../help/licence-and-privacy.md)). After that, `DoorPlusProvider.get()`
    throws. Do not keep `Door`, `Trigger` or `DoorGroup` objects across a Door+ restart: get them again.

The entry point gives access to five services:

| Accessor | Service | Purpose |
|---|---|---|
| `api.doors()` | `DoorManager` | Find, move, create and delete doors |
| `api.access()` | `AccessManager` | Roles, access checks, access entries |
| `api.triggers()` | `TriggerManager` | Find, add, remove and activate triggers |
| `api.groups()` | `GroupManager` | Find door groups and operate all their doors |
| `api.hooks()` | `HookRegistry` | Protection hooks, custom block and NPC providers, see [Extension points](extensions.md) |

## Threading rules

1. Call every method from the **main server thread**, unless its Javadoc says otherwise.
2. These methods throw `IllegalStateException` when called from another thread: `toggle`, `create`, `delete`,
   `grant`, `revoke`, `canUse`, `TriggerManager.add`, `remove`, `activate`, `GroupManager.toggle`, and every `Door`
   setter.
3. Lookups read in-memory caches and may be called from any thread: `byId`, `byName`, `ownedBy`, `all`, `at`,
   `inChunk` and `of`.
4. Methods that may wait return a `CompletableFuture`. It is always completed on the main thread, so you can use
   the Bukkit API inside `thenAccept` without scheduling a task.
5. All events are fired synchronously on the main thread, see [Events](events.md).
6. Hooks and providers you register are called on the main thread and must be fast.

!!! tip "Unwrapping failures"
    A failed future may carry its exception directly or wrapped in a `CompletionException`. The examples on this
    page use this helper to get the real cause:

    ```java
    static Throwable cause(Throwable error) {
        return error instanceof java.util.concurrent.CompletionException && error.getCause() != null
                ? error.getCause() : error;
    }
    ```

## Live views

`Door`, `Trigger` and `DoorGroup` are **live views** of the plugin state, not copies: their getters always return
the current values. Pass back to the API only objects you obtained from it. An object implemented by your own code
is rejected with `IllegalArgumentException`.

## Doors

### Finding doors

```java
import io.github.cristiaaaaan.doorplus.api.DoorPlusProvider;
import io.github.cristiaaaaan.doorplus.api.door.Door;
import io.github.cristiaaaaan.doorplus.api.door.DoorManager;
import org.bukkit.block.Block;
import org.bukkit.entity.Player;

DoorManager doors = DoorPlusProvider.get().doors();

doors.byId(42).ifPresent(door -> player.sendMessage("Door #42 is " + door.getName()));

// Name is case-insensitive and unique per owner
doors.byName(player.getUniqueId(), "castle_gate")
     .ifPresent(door -> player.sendMessage("Open: " + door.isOpen()));

// The door whose current blocks contain this block
Block target = player.getTargetBlockExact(5);
if (target != null) {
    doors.at(target.getLocation()).ifPresent(door ->
            player.sendMessage("You are looking at " + door.getName() + " (" + door.getType().id() + ")"));
}

int owned = doors.ownedBy(player.getUniqueId()).size();
```

| `DoorManager` method | Returns |
|---|---|
| `byId(long id)` | `Optional<Door>` |
| `byName(UUID owner, String name)` | `Optional<Door>`, name case-insensitive |
| `ownedBy(UUID owner)` | `Collection<Door>` |
| `all()` | `Collection<Door>` (doors of this server) |
| `at(Location blockLocation)` | `Optional<Door>` whose current occupancy contains the block; `IllegalArgumentException` for a location without world |
| `inChunk(World world, int chunkX, int chunkZ)` | `Collection<Door>` whose current occupancy touches the chunk |

Useful `Door` getters:

| Getter | Meaning |
|---|---|
| `getId()`, `getName()`, `getType()`, `getOwner()`, `getWorldId()` | Identity |
| `getBaseRegion()` | `Cuboid` occupied in state 0 (closed) |
| `getOccupancy()` | `List<Cuboid>` occupied in the current state |
| `getState()` | Binary types: 0 closed, 1 open. Platform: index of the current stop. Windmill, flag, clock: 0 stopped, 1 running |
| `isOpen()` | Binary types in state 1, perpetual types running, platforms away from stop 0 |
| `isMoving()` | An animation is running |
| `getOpenDirection()`, `getPrivacy()`, `isLocked()`, `isRedstoneAllowed()`, `areBlocksProtected()`, `getAutoCloseSeconds()`, `getSpeedMultiplier()` | Settings |
| `getAccessEntries()` | Access entries, owner excluded |
| `getTriggers()` | Triggers whose target is this door |

`DoorType` lists the 13 types (`BIG_DOOR`, `DRAWBRIDGE`, `PORTCULLIS`, `SLIDING_DOOR`, `GARAGE_DOOR`, `SPLIT_DOOR`,
`FOLDING_DOOR`, `ROLLING_SHUTTER`, `PLATFORM`, `REVOLVING_DOOR`, `WINDMILL`, `FLAG`, `CLOCK`). `type.id()` gives the
lower-case id used in permissions and config (`big_door`), `type.category()` gives `BINARY`, `PLATFORM`, `REVOLVING`
or `PERPETUAL`, and `type.isPerpetual()` is true for windmill, flag and clock.

### Changing settings

`Door` setters apply the change, save it and, for privacy, fire `DoorPrivacyChangeEvent`. Call them on the main
thread.

```java
import io.github.cristiaaaaan.doorplus.api.door.Privacy;

door.setPrivacy(Privacy.PUBLIC);   // a listener may cancel it: the call then does nothing
door.setLocked(true);
door.setAutoCloseSeconds(10);      // IllegalArgumentException outside 0..limits.max-auto-close-seconds
door.setSpeedMultiplier(1.5);      // IllegalArgumentException outside limits.speed-multiplier
```

### Moving a door

`toggle(Door, ToggleOptions)` asks Door+ to move a door, with cause `API`. It returns a
`CompletableFuture<ToggleOutcome>` completed on the main thread when the movement **starts** or is refused (not when
it ends: listen to `DoorToggleEndEvent` for that).

Build the options from a factory, then add modifiers. Each modifier returns a new object.

| Factory | Action |
|---|---|
| `ToggleOptions.toggle()` | Invert the state |
| `ToggleOptions.open()` | Move towards open |
| `ToggleOptions.close()` | Move towards closed |
| `ToggleOptions.gotoStop(int stop)` | Send a platform to a stop (first stop is 0); `IllegalArgumentException` for a negative stop |

| Modifier | Effect |
|---|---|
| `actor(UUID actor)` | Player responsible for the movement. With an actor, the access is checked |
| `checkAccess(boolean check)` | `false` skips the access check even with an actor |
| `instant()` | Move without animation |

Access rules:

- Without an actor, the access is **not** checked.
- With an actor, the access is checked (privacy, role, key) unless you call `checkAccess(false)`. The actor must be
  online, otherwise the result is `DENIED`.
- The lock is always honoured: a locked door gives `LOCKED`.

```java
import io.github.cristiaaaaan.doorplus.api.DoorPlusProvider;
import io.github.cristiaaaaan.doorplus.api.door.ToggleOptions;
import io.github.cristiaaaaan.doorplus.api.door.ToggleOutcome;
import io.github.cristiaaaaan.doorplus.api.door.ToggleResult;

DoorPlusProvider.get().doors()
        .toggle(door, ToggleOptions.open().actor(player.getUniqueId()))
        .thenAccept(outcome -> {
            switch (outcome.result()) {
                case SUCCESS -> player.sendMessage("The door is opening.");
                case NO_CHANGE -> player.sendMessage("The door is already open.");
                case OBSTRUCTED -> player.sendMessage("Something is in the way at " + outcome.where());
                default -> player.sendMessage("Refused: " + outcome.result());
            }
        });

// A server-side action: no actor, no access check, no animation
DoorPlusProvider.get().doors().toggle(door, ToggleOptions.close().instant());
```

`ToggleOutcome` is a record with `result()` and `where()`. `where()` is the offending block for `BLOCKED_SOURCE` and
`OBSTRUCTED`, otherwise `null`. `result().isSuccess()` is true only for `SUCCESS`.

| `ToggleResult` | Meaning |
|---|---|
| `SUCCESS` | The movement started |
| `NO_CHANGE` | The door is already in the requested state |
| `DOOR_NOT_FOUND` | The door was deleted |
| `WORLD_NOT_LOADED` | The world of the door is not loaded |
| `PAUSED` | Movements are paused by an administrator (`/dp admin pause`) |
| `LOCKED` | The door is locked |
| `DENIED` | Access denied, or the actor is offline while the access is checked |
| `BUSY` | An animation is already running |
| `COOLDOWN` | The door cooldown (`movement.toggle-cooldown-ticks`) has not expired |
| `BLOCKED_SOURCE` | A block in the door cannot be animated; see `where()` |
| `OBSTRUCTED` | The destination is obstructed; see `where()` |
| `OUT_OF_WORLD` | The destination is outside the world height limits |
| `PROTECTED` | A protection hook denied the movement |
| `CANCELLED` | A plugin cancelled `DoorPreToggleEvent` |
| `ERROR` | Unexpected exception; details in the console |

`toggle` throws `IllegalArgumentException` for a `Door` not obtained from Door+, and `IllegalStateException` off the
main thread.

### Creating a door

`create(DoorSpec)` creates a door with the same validations as the guided creation (shape, blocks, overlaps,
`DoorCreateEvent`). The future fails with `IllegalArgumentException` whose message is the reason, for example
`The region overlaps the door gate.`

`DoorSpec` is a record. `DoorSpec.of(type, name, owner, worldId, region)` fills only the mandatory fields; add the
rest with `withPivot(BlockPos, Axis)`, `withDirection(OpenDirection)`, `withBlocksToMove(int)` and
`withTypeData(Map<String, Object>)`. The name must match `^[A-Za-z0-9_-]{1,32}$` and be unique for the owner. A
`null` direction or 0 blocks to move means "type default". Unknown `typeData` keys are ignored.

```java
import io.github.cristiaaaaan.doorplus.api.DoorPlusProvider;
import io.github.cristiaaaaan.doorplus.api.door.DoorSpec;
import io.github.cristiaaaaan.doorplus.api.door.DoorType;
import io.github.cristiaaaaan.doorplus.api.door.OpenDirection;
import io.github.cristiaaaaan.doorplus.api.geometry.BlockPos;
import io.github.cristiaaaaan.doorplus.api.geometry.Cuboid;

// A sliding door 5 wide and 4 high, one block thick along Z, opening towards east
Cuboid region = Cuboid.of(new BlockPos(100, 64, 200), new BlockPos(104, 67, 200));
DoorSpec spec = DoorSpec.of(DoorType.SLIDING_DOOR, "shop_gate", player.getUniqueId(),
                player.getWorld().getUID(), region)
        .withDirection(OpenDirection.EAST);

DoorPlusProvider.get().doors().create(spec).whenComplete((door, error) -> {
    if (error != null) {
        player.sendMessage("Cannot create the door: " + cause(error).getMessage());
    } else {
        player.sendMessage("Created door #" + door.getId());
    }
});
```

!!! note
    The pivot and the type parameters each type needs are described on the [door type pages](../doors/types/index.md).
    The geometry helpers `Cuboid` and `BlockPos` are immutable records (`Cuboid.of(a, b)` sorts the corners for
    you; `BlockPos.of(Block)` and `BlockPos.of(Location)` convert from Bukkit).

### Deleting a door

`delete(Door)` deletes the door and leaves its blocks where they are. It fires `DoorDeleteEvent`. The future fails
with `IllegalArgumentException` if the door no longer exists, and with `IllegalStateException` if the deletion was
cancelled by a listener or refused (for example an open rolling shutter that cannot close because it is obstructed).

```java
DoorPlusProvider.get().doors().delete(door).whenComplete((ignored, error) -> {
    if (error != null) getLogger().warning("Delete failed: " + cause(error).getMessage());
});
```

## Access

`AccessManager` answers "who may use this door" and edits the access list. Roles are `USER`, `ADMIN` and `OWNER`,
in increasing order (`role.atLeast(Role.ADMIN)`); see [Privacy and roles](../access/privacy-and-roles.md) for what
each one may do.

| Method | Purpose |
|---|---|
| `roleOf(Player, Door)` | Role of an online player, `null` if none. Bypass permissions are not considered |
| `roleOf(UUID, Door)` | Role of a possibly offline player, as a future completed on the main thread. Group and permission entries are resolved through LuckPerms when installed; otherwise only player entries count |
| `canUse(Player, Door)` | Whether the player may use the door now (privacy, role, key) |
| `canUse(Player, Door, Trigger)` | Same, through a trigger: the trigger access override applies. `IllegalArgumentException` if the trigger does not operate that door |
| `grant(Door, AccessSubject, Role)` | Add or update an entry; fires `DoorAccessChangeEvent` |
| `revoke(Door, AccessSubject)` | Remove an entry if present; fires `DoorAccessChangeEvent` |

Subjects are built with `AccessSubject.player(UUID)`, `AccessSubject.group(String)` (a LuckPerms group, stored
lower-case) and `AccessSubject.permission(String)` (a node matching `^[a-z0-9._*-]{1,128}$`). The factories throw
`IllegalArgumentException` for an invalid group name or node.

```java
import io.github.cristiaaaaan.doorplus.api.DoorPlusProvider;
import io.github.cristiaaaaan.doorplus.api.access.AccessManager;
import io.github.cristiaaaaan.doorplus.api.access.AccessSubject;
import io.github.cristiaaaaan.doorplus.api.access.Role;

AccessManager access = DoorPlusProvider.get().access();

access.grant(door, AccessSubject.player(friend.getUniqueId()), Role.USER);
access.grant(door, AccessSubject.group("vip"), Role.USER);
access.grant(door, AccessSubject.permission("myplugin.guard"), Role.ADMIN);
access.revoke(door, AccessSubject.group("vip"));

if (access.canUse(player, door)) {
    player.sendMessage("You may use " + door.getName());
}

access.roleOf(offlinePlayerId, door).thenAccept(role ->
        getLogger().info("Role: " + (role == null ? "none" : role.name())));
```

`grant` throws `IllegalArgumentException` for `Role.OWNER` (transfer the door instead), for the owner as subject, or
when the access list is full (`limits.max-access-entries`). A grant or revoke cancelled by a listener does nothing
and does not throw.

## Triggers

`TriggerManager` finds, adds, removes and activates triggers. A `Trigger` is a read-only live view: type, provider,
position, action, access mode, cooldown and so on.

| Method | Purpose |
|---|---|
| `byId(long id)` | `Optional<Trigger>` |
| `of(Door)`, `of(DoorGroup)` | Triggers targeting a door or a group |
| `at(Block)` | Block, redstone and custom-block triggers at the block |
| `at(Entity)` | Entity and furniture triggers bound to the entity |
| `add(TriggerSpec)` | Add a trigger. Validates and fires `TriggerAddEvent`. Fails with `IllegalArgumentException` whose message is the reason code (`DUPLICATE`, `LIMIT`, `INSIDE_DOOR`, `INVALID_ACTION`, ...) |
| `remove(Trigger)` | Remove with cause `API`; completes immediately if already removed |
| `activate(Trigger, Player)` | Act as if the source fired; completes with the outcome (of the first door for a group target) |

`TriggerSpec.block(target, type, worldId, position, action)` builds a vanilla block or redstone trigger with the
defaults (access mode `INHERIT`, default cooldown). Add `withAccessMode(TriggerAccessMode)` and
`withCreator(UUID)` if needed. For entity, NPC, custom block, furniture and proximity triggers use the full record
constructor.

```java
import io.github.cristiaaaaan.doorplus.api.DoorPlusProvider;
import io.github.cristiaaaaan.doorplus.api.geometry.BlockPos;
import io.github.cristiaaaaan.doorplus.api.trigger.TriggerAccessMode;
import io.github.cristiaaaaan.doorplus.api.trigger.TriggerAction;
import io.github.cristiaaaaan.doorplus.api.trigger.TriggerSpec;
import io.github.cristiaaaaan.doorplus.api.trigger.TriggerTarget;
import io.github.cristiaaaaan.doorplus.api.trigger.TriggerType;

// A button that toggles the door for everyone
TriggerSpec spec = TriggerSpec.block(TriggerTarget.door(door.getId()), TriggerType.BLOCK,
                button.getWorld().getUID(), BlockPos.of(button), TriggerAction.TOGGLE)
        .withAccessMode(TriggerAccessMode.PUBLIC)
        .withCreator(player.getUniqueId());

DoorPlusProvider.get().triggers().add(spec).whenComplete((trigger, error) -> {
    if (error != null) player.sendMessage("Not linked: " + cause(error).getMessage());
    else player.sendMessage("Linked trigger #" + trigger.getId());
});
```

!!! note "API calls have no player"
    `add` runs without a player: the permission, role, maximum distance and protection hook checks that apply to
    players are skipped. The world, door-trigger limit (`limits.max-triggers-per-door`), duplicate, "inside a door",
    entity type and action checks still apply.

## Groups

A `DoorGroup` is a named set of doors operated together (`getId()`, `getName()`, `getOwner()`, `getDoors()`,
`getCreatedAt()`). `GroupManager` offers `byId`, `byName(UUID owner, String name)`, `ownedBy(UUID owner)` and
`toggle(DoorGroup, ToggleOptions)`.

`toggle` operates every door of the group and completes with one `ToggleOutcome` per door. With
`ToggleOptions.toggle()` it opens all the doors if at least one binary door is closed, otherwise it closes all. When
the access is checked the actor must be online, otherwise every door is `DENIED`.

```java
import io.github.cristiaaaaan.doorplus.api.DoorPlusProvider;
import io.github.cristiaaaaan.doorplus.api.door.ToggleOptions;
import io.github.cristiaaaaan.doorplus.api.door.ToggleOutcome;

DoorPlusProvider.get().groups().byName(player.getUniqueId(), "castle").ifPresent(group ->
        DoorPlusProvider.get().groups().toggle(group, ToggleOptions.open().actor(player.getUniqueId()))
                .thenAccept(outcomes -> {
                    long moving = outcomes.stream().filter(o -> o.result().isSuccess()).count();
                    player.sendMessage(moving + "/" + outcomes.size() + " doors moving");
                }));
```

## Error summary

| Situation | What you get |
|---|---|
| State-changing call off the main thread | `IllegalStateException` thrown |
| `Door`, `Trigger` or `DoorGroup` not obtained from Door+ | `IllegalArgumentException` thrown |
| Invalid input to `create`, `TriggerManager.add`, `grant` | `IllegalArgumentException` (thrown, or as the future's failure) |
| Deleted door passed to `toggle` | Outcome `DOOR_NOT_FOUND` |
| Refused or cancelled `delete` | Future fails with `IllegalStateException` |
| Door+ not enabled | `DoorPlusProvider.get()` throws `IllegalStateException` |

## See also

- [Events](events.md): react to and cancel door actions.
- [Extension points](extensions.md): protection hooks, custom block and NPC providers.
- The Javadoc ships with the artifact (sources and Javadoc jars are published on JitPack).
