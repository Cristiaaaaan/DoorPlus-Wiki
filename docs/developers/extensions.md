# Extension points

This page is for plugin developers who want to connect another plugin to Door+: a protection or claims plugin, a
custom blocks and furniture plugin, or an NPC plugin. It explains the three interfaces of `HookRegistry`, their
lifecycle, and gives a minimal example of each. For the dependency and how to get the API, see
[Developer API](api.md).

The built-in integrations (WorldGuard, ItemsAdder, Nexo, Oraxen, CraftEngine, Citizens, FancyNpcs) use exactly these
interfaces, so a plugin you connect yourself works like them.

## HookRegistry

Get it with `DoorPlusProvider.get().hooks()`. It has three methods:

| Method | Registers | Fails when |
|---|---|---|
| `registerProtectionHook(Plugin owner, ProtectionHook hook)` | A protection hook | never |
| `registerContentProvider(Plugin owner, CustomContentProvider provider)` | A custom blocks / furniture / items provider | `IllegalArgumentException` if a provider with the same `id()` is already registered |
| `registerNpcProvider(Plugin owner, NpcProvider provider)` | An NPC provider | `IllegalArgumentException` if a provider with the same `id()` is already registered |

`owner` is your plugin. Door+ logs one line per registration, for example
`Protection hook registered: SpawnGuard (MyPlugin)`, and lists every hook and provider in `/dp admin version`
(line **Hooks and providers**, as `protection:<name>`, `content:<id>`, `npc:<id>`).

### Lifecycle

1. Register in your `onEnable`, after Door+ is enabled (`depend: [DoorPlus]` or `softdepend: [DoorPlus]` in your
   `plugin.yml`).
2. Registrations are removed **automatically when your plugin is disabled**. You do not need to unregister.
3. Registrations belong to the running Door+ instance. If Door+ itself is disabled and enabled again (for example
   with a plugin manager), register again. You can listen to `PluginEnableEvent` for `DoorPlus`:

    ```java
    import org.bukkit.event.server.PluginEnableEvent;

    @EventHandler
    public void onPluginEnable(PluginEnableEvent event) {
        if (event.getPlugin().getName().equals("DoorPlus")) registerDoorPlusHooks();
    }
    ```

4. All methods of your hooks and providers are called on the main thread. They must be fast: `canMove`, for
   example, runs for every movement of every door.

## Protection hooks

A `ProtectionHook` lets a protection plugin forbid creating doors, moving doors and linking triggers. **Every**
registered hook must allow an operation. The name of the first hook that denies it is shown to the player. An
exception thrown by a hook is logged (`Protection hook <name> threw; treated as denial`) and counts as a denial.

| Method | Called when | Arguments | Player message when it returns `false` |
|---|---|---|---|
| `name()` | A message needs the hook name | | |
| `canCreate(Player, World, Collection<Cuboid> regions)` | A player creates a door or redefines its region | Base region and the door's open position | **You cannot create a door here (denied by &lt;name&gt;).** |
| `canMove(Door, Collection<Cuboid> target)` | Every movement, before `DoorPreToggleEvent` | Cuboids the door will occupy at the destination | **&lt;door&gt; cannot move here: denied by &lt;name&gt;.** |
| `canLinkTrigger(Player, Location source)` | A player links a trigger | Position of the trigger source | **The protection &lt;name&gt; does not allow you to link this source.** |

- Players with `doorplus.admin.bypass.protection` skip `canCreate` and `canLinkTrigger`. `canMove` is always
  checked.
- `canCreate` and `canLinkTrigger` are only called when a player is involved: doors created and triggers added
  through the API do not go through them.
- A refused movement gives the result `PROTECTED` to API callers.

### Example

Forbid doors and triggers inside an arena, and forbid any door from moving into it.

```java
import io.github.cristiaaaaan.doorplus.api.door.Door;
import io.github.cristiaaaaan.doorplus.api.geometry.BlockPos;
import io.github.cristiaaaaan.doorplus.api.geometry.Cuboid;
import io.github.cristiaaaaan.doorplus.api.hook.ProtectionHook;
import java.util.Collection;
import java.util.UUID;
import org.bukkit.Location;
import org.bukkit.World;
import org.bukkit.entity.Player;
import org.jetbrains.annotations.NotNull;

public final class ArenaProtectionHook implements ProtectionHook {

    private final UUID arenaWorld;
    private final Cuboid arena;

    public ArenaProtectionHook(World world, Cuboid arena) {
        this.arenaWorld = world.getUID();
        this.arena = arena;
    }

    @Override
    public @NotNull String name() {
        return "Arena";
    }

    @Override
    public boolean canCreate(@NotNull Player player, @NotNull World world, @NotNull Collection<Cuboid> regions) {
        return !touchesArena(world.getUID(), regions);
    }

    @Override
    public boolean canMove(@NotNull Door door, @NotNull Collection<Cuboid> target) {
        return !touchesArena(door.getWorldId(), target);
    }

    @Override
    public boolean canLinkTrigger(@NotNull Player player, @NotNull Location source) {
        World world = source.getWorld();
        return world == null || !world.getUID().equals(arenaWorld) || !arena.contains(BlockPos.of(source));
    }

    private boolean touchesArena(UUID world, Collection<Cuboid> cuboids) {
        if (!world.equals(arenaWorld)) return false;
        for (Cuboid cuboid : cuboids) {
            if (cuboid.intersects(arena)) return true;
        }
        return false;
    }
}
```

Register it:

```java
DoorPlusProvider.get().hooks().registerProtectionHook(this,
        new ArenaProtectionHook(arenaWorld, Cuboid.of(new BlockPos(-50, 0, -50), new BlockPos(50, 120, 50))));
```

## Custom content providers

A `CustomContentProvider` connects a plugin that adds custom blocks, furniture and items. With it, players can use
your custom blocks and furniture as [triggers](../access/triggers.md) (types `CUSTOM_BLOCK` and `FURNITURE`) and your
custom items as [keys](../access/keys.md). Players link them like vanilla sources: `/dp trigger add <door> block`
for a custom block, `/dp trigger add <door> entity` for a furniture.

| Method | What Door+ expects |
|---|---|
| `id()` | Short, stable, lower-case id stored in triggers and keys, e.g. `mycontent`. The built-in ids are `itemsadder`, `nexo`, `oraxen`, `craftengine` |
| `isReady()` | `false` until your plugin has loaded its data. While it is `false`, Door+ does not recognise your objects when a player selects a trigger source, and does not remove triggers whose id seems to have changed |
| `customBlockId(Block)` | Id of an already placed custom block, or empty |
| `furnitureId(Entity baseEntity)` | Id of an already spawned furniture, from its **base entity**, or empty |
| `itemId(ItemStack)` | Id of a custom item, or empty. Door+ stores keys as `<provider id>:<item id>` |
| `bind(ContentListener)` | Door+ gives you its single listener. Keep it and forward your plugin's events to it |

`bind` is called once when you register (if Door+'s trigger system is already running, which is the normal case
when you register in `onEnable`). Ignore events until you have a listener.

The `ContentListener` you receive has four methods. Each returns `true` when you must **cancel** your plugin's
event (for example because the click was used to link or activate a trigger, or because the block is a protected
trigger source):

| Method | Call it when |
|---|---|
| `onBlockInteract(Player, Block, String id)` | A player right-clicks one of your custom blocks |
| `onFurnitureInteract(Player, Entity baseEntity, String id)` | A player right-clicks one of your furniture |
| `onBlockBreak(Player, Block, String id)` | One of your custom blocks is broken; the player is `null` if nobody broke it (explosion, ...) |
| `onFurnitureBreak(Player, Entity baseEntity, String id)` | One of your furniture is removed; the player may be `null` |

Rules:

- Always pass the **base entity** of a furniture, not the hitbox entity the player clicked.
- If your plugin fires its click event twice (main hand and off hand), forward only one of them.

### Example

A plugin whose custom blocks are player heads and whose furniture are `Interaction` entities, both tagged with an
id in their persistent data, and whose items carry the same tag.

```java
import io.github.cristiaaaaan.doorplus.api.hook.ContentListener;
import io.github.cristiaaaaan.doorplus.api.hook.CustomContentProvider;
import java.util.Optional;
import org.bukkit.NamespacedKey;
import org.bukkit.block.Block;
import org.bukkit.block.TileState;
import org.bukkit.entity.Entity;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.block.Action;
import org.bukkit.event.block.BlockBreakEvent;
import org.bukkit.event.player.PlayerInteractEntityEvent;
import org.bukkit.event.player.PlayerInteractEvent;
import org.bukkit.inventory.EquipmentSlot;
import org.bukkit.inventory.ItemStack;
import org.bukkit.persistence.PersistentDataType;
import org.bukkit.plugin.Plugin;
import org.jetbrains.annotations.NotNull;
import org.jetbrains.annotations.Nullable;

public final class MyContentProvider implements CustomContentProvider, Listener {

    private final NamespacedKey key;
    private volatile @Nullable ContentListener listener;

    public MyContentProvider(Plugin plugin) {
        this.key = new NamespacedKey(plugin, "content_id");
    }

    @Override
    public @NotNull String id() {
        return "mycontent";
    }

    @Override
    public boolean isReady() {
        return true; // return false until your data is loaded
    }

    @Override
    public @NotNull Optional<String> customBlockId(@NotNull Block block) {
        if (!(block.getState() instanceof TileState state)) return Optional.empty();
        return Optional.ofNullable(state.getPersistentDataContainer().get(key, PersistentDataType.STRING));
    }

    @Override
    public @NotNull Optional<String> furnitureId(@NotNull Entity baseEntity) {
        return Optional.ofNullable(baseEntity.getPersistentDataContainer().get(key, PersistentDataType.STRING));
    }

    @Override
    public @NotNull Optional<String> itemId(@NotNull ItemStack item) {
        if (!item.hasItemMeta()) return Optional.empty();
        return Optional.ofNullable(item.getItemMeta().getPersistentDataContainer().get(key, PersistentDataType.STRING));
    }

    @Override
    public void bind(@NotNull ContentListener listener) {
        this.listener = listener;
    }

    @EventHandler
    public void onClick(PlayerInteractEvent event) {
        ContentListener l = listener;
        Block block = event.getClickedBlock();
        if (l == null || block == null || event.getAction() != Action.RIGHT_CLICK_BLOCK
                || event.getHand() != EquipmentSlot.HAND) return;
        customBlockId(block).ifPresent(id -> {
            if (l.onBlockInteract(event.getPlayer(), block, id)) event.setCancelled(true);
        });
    }

    @EventHandler(ignoreCancelled = true)
    public void onBreak(BlockBreakEvent event) {
        ContentListener l = listener;
        if (l == null) return;
        customBlockId(event.getBlock()).ifPresent(id -> {
            if (l.onBlockBreak(event.getPlayer(), event.getBlock(), id)) event.setCancelled(true);
        });
    }

    @EventHandler(ignoreCancelled = true)
    public void onFurnitureClick(PlayerInteractEntityEvent event) {
        ContentListener l = listener;
        if (l == null || event.getHand() != EquipmentSlot.HAND) return;
        Entity base = event.getRightClicked();
        furnitureId(base).ifPresent(id -> {
            if (l.onFurnitureInteract(event.getPlayer(), base, id)) event.setCancelled(true);
        });
    }

    /** Call this from your own code before removing a furniture; returns false if Door+ asks to keep it. */
    public boolean beforeFurnitureRemoved(@Nullable org.bukkit.entity.Player player, Entity base) {
        ContentListener l = listener;
        Optional<String> id = furnitureId(base);
        return l == null || id.isEmpty() || !l.onFurnitureBreak(player, base, id.get());
    }
}
```

Register it:

```java
MyContentProvider provider = new MyContentProvider(this);
getServer().getPluginManager().registerEvents(provider, this);
DoorPlusProvider.get().hooks().registerContentProvider(this, provider);
```

## NPC providers

An `NpcProvider` connects an NPC plugin, so that players can use NPCs as [triggers](../access/triggers.md) (type
`NPC`). Players link an NPC with `/dp trigger add <door> entity` and a right click on the NPC.

| Method | What Door+ expects |
|---|---|
| `id()` | Short, stable, lower-case id stored in triggers, e.g. `mynpcs`. The built-in ids are `citizens` and `fancynpcs` |
| `exists(String npcId)` | Whether the NPC exists. Door+ uses it to recognise the NPC when a player links it |
| `location(String npcId)` | Current location of the NPC, or empty. Stored as the trigger position |
| `bind(NpcListener)` | Door+ gives you its single listener. Keep it and forward clicks and removals to it |

NPC ids **must stay the same across restarts**: Door+ saves them in the trigger.

The `NpcListener` you receive has two methods:

| Method | Call it when |
|---|---|
| `onInteract(Player, String npcId)` | A player right-clicks an NPC. Returns `true` if you must cancel your plugin's click event |
| `onRemove(String npcId)` | The NPC was **permanently** deleted. Door+ then removes its triggers (cause `SOURCE_REMOVED`). Do not call it when NPCs are merely unloaded, for example at shutdown |

Clicks are forwarded through `onInteract` for both linking and activation, so NPCs that are not Bukkit entities
work too.

### Example

An NPC plugin that keeps its NPCs in a map from a stable id to the entity UUID.

```java
import io.github.cristiaaaaan.doorplus.api.hook.NpcListener;
import io.github.cristiaaaaan.doorplus.api.hook.NpcProvider;
import java.util.Map;
import java.util.Optional;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;
import org.bukkit.Bukkit;
import org.bukkit.Location;
import org.bukkit.entity.Entity;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.player.PlayerInteractEntityEvent;
import org.bukkit.inventory.EquipmentSlot;
import org.jetbrains.annotations.NotNull;
import org.jetbrains.annotations.Nullable;

public final class MyNpcProvider implements NpcProvider, Listener {

    /** Stable NPC id -> current entity UUID, maintained by your NPC plugin. */
    private final Map<String, UUID> npcs = new ConcurrentHashMap<>();
    private volatile @Nullable NpcListener listener;

    @Override
    public @NotNull String id() {
        return "mynpcs";
    }

    @Override
    public boolean exists(@NotNull String npcId) {
        return npcs.containsKey(npcId);
    }

    @Override
    public @NotNull Optional<Location> location(@NotNull String npcId) {
        UUID entityId = npcs.get(npcId);
        Entity entity = entityId == null ? null : Bukkit.getEntity(entityId);
        return entity == null ? Optional.empty() : Optional.of(entity.getLocation());
    }

    @Override
    public void bind(@NotNull NpcListener listener) {
        this.listener = listener;
    }

    @EventHandler(ignoreCancelled = true)
    public void onClick(PlayerInteractEntityEvent event) {
        NpcListener l = listener;
        if (l == null || event.getHand() != EquipmentSlot.HAND) return;
        UUID clicked = event.getRightClicked().getUniqueId();
        npcs.forEach((npcId, entityId) -> {
            if (entityId.equals(clicked) && l.onInteract(event.getPlayer(), npcId)) event.setCancelled(true);
        });
    }

    /** Call from your plugin when an NPC is created or respawned with a new entity. */
    public void track(String npcId, Entity entity) {
        npcs.put(npcId, entity.getUniqueId());
    }

    /** Call from your plugin when an NPC is deleted for good. */
    public void deleted(String npcId) {
        npcs.remove(npcId);
        NpcListener l = listener;
        if (l != null) l.onRemove(npcId);
    }
}
```

Register it:

```java
MyNpcProvider provider = new MyNpcProvider();
getServer().getPluginManager().registerEvents(provider, this);
DoorPlusProvider.get().hooks().registerNpcProvider(this, provider);
```

!!! tip
    Register your listener at a priority that runs before other plugins act on the click, and cancel the event
    when Door+ returns `true`, so that the click is not also handled as a normal interaction.
