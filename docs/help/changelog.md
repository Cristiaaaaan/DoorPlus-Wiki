# Changelog

This page lists what changed in each Door+ release, newest first. It is useful to server owners deciding whether
to update.

## 1.0.0

First release (API 1.0.0).

- 13 door types: [big door](../doors/types/big-door.md), [drawbridge](../doors/types/drawbridge.md),
  [portcullis](../doors/types/portcullis.md), [sliding door](../doors/types/sliding-door.md),
  [garage door](../doors/types/garage-door.md), [split door](../doors/types/split-door.md),
  [folding door](../doors/types/folding-door.md), [rolling shutter](../doors/types/rolling-shutter.md),
  [platform](../doors/types/platform.md), [revolving door](../doors/types/revolving-door.md),
  [windmill](../doors/types/windmill.md), [flag](../doors/types/flag.md), [clock](../doors/types/clock.md).
- Animation engines `block_display` (default) and `falling_block`; collision modes `none`, `shulker`, `push`
  ([Movement and animation](../doors/movement.md)).
- Doors move through passable blocks (barrier, light, structure void) and restore them.
- Privacy (private / restricted / public), roles (owner / admin / user), access entries for players, LuckPerms
  groups and permissions; optional per-door access for the members and owners of the WorldGuard region around
  the door ([Privacy and roles](../access/privacy-and-roles.md)).
- Multiple triggers per door: blocks, redstone, entities, proximity, ItemsAdder / Nexo / Oraxen / CraftEngine
  custom blocks and furniture, Citizens / FancyNpcs NPCs; per-trigger action and access override
  ([Triggers](../access/triggers.md)).
- [Keys](../access/keys.md), [door groups](../access/groups.md), block protection, auto-close, per-door speed,
  vanilla sounds.
- Inventory [menus](../doors/menus.md) and [commands](../reference/commands.md); English and Italian
  [language files](../reference/language.md).
- SQLite and MySQL/MariaDB [storage](../reference/storage.md) with migrations; anti-crash journal.
- [PlaceholderAPI](../integrations/placeholderapi.md) expansion, [WorldGuard](../integrations/worldguard.md) hook,
  public [API](../developers/api.md) on JitPack.
- [Import from BigDoors](../getting-started/bigdoors-import.md) v1.
- Sold on BuiltByBit and Voxel Shop: release builds verify the purchase with the marketplace (7-day grace
  period when it cannot be reached) and are obfuscated; the public API stays readable
  ([Licence, verification and privacy](licence-and-privacy.md)).
- Anonymous usage statistics on bStats (can be turned off in `plugins/bStats/config.yml`).
