# Integrations

This page lists the plugins Door+ works with, what each one adds, and how to check which integrations are active
on your server. It is for server administrators.

All integrations are optional. Door+ works on its own; when one of these plugins is installed, Door+ connects to it
automatically at startup. You do not need to download any add-on.

## Supported plugins

| Plugin | What it adds | Page |
|---|---|---|
| LuckPerms | Access entries for LuckPerms groups, with group lists in menus and tab completion | [LuckPerms](luckperms.md) |
| WorldGuard | Build checks when creating doors and linking triggers, the flags `doorplus-create` and `doorplus-toggle`, region access for doors | [WorldGuard](worldguard.md) |
| PlaceholderAPI | The `%doorplus_...%` placeholders | [PlaceholderAPI](placeholderapi.md) |
| ItemsAdder | Custom blocks and furniture as triggers, custom items as keys | [Custom blocks and furniture](custom-blocks.md) |
| Nexo | Custom blocks and furniture as triggers, custom items as keys | [Custom blocks and furniture](custom-blocks.md) |
| Oraxen | Custom blocks and furniture as triggers, custom items as keys | [Custom blocks and furniture](custom-blocks.md) |
| CraftEngine | Custom blocks and furniture as triggers, custom items as keys | [Custom blocks and furniture](custom-blocks.md) |
| Citizens | NPCs as triggers | [NPCs](npcs.md) |
| FancyNpcs | NPCs as triggers | [NPCs](npcs.md) |

Door+ 1.0.0 was built against these versions of the plugin APIs: LuckPerms 5.5, WorldGuard 7.0.17 (WorldEdit
7.3.18), PlaceholderAPI 2.12.3, ItemsAdder 4.0.17, Nexo 1.30.2, Oraxen 1.220.1, CraftEngine 26.10, Citizens 2.0.44,
FancyNpcs 2.9.2. LuckPerms, WorldGuard (with FastAsyncWorldEdit), PlaceholderAPI, ItemsAdder and FancyNpcs were
also tested on a live Paper 1.21.8 server.

Other plugins can add their own protection checks, custom block sources and NPC sources through the Door+ API:
see [Extension points](../developers/extensions.md).

## Detection at startup

When Door+ starts, it checks each plugin in the table. For each one it writes one line in the console:

| Console line | Meaning |
|---|---|
| `<Plugin> <version>: active (...)` | The integration works. The brackets say what was enabled, e.g. `WorldGuard 7.0.17: active (protection hook, region access, flags doorplus-create, doorplus-toggle)`. |
| `<Plugin>: not installed; ... unavailable` | The plugin is missing or not enabled. |
| `<Plugin>: disabled in config (integrations.<name>); ... off` | You switched the integration off in `config.yml`. |
| `<Plugin>: hook failed, integration disabled` | The plugin is installed but Door+ could not connect to it (usually an incompatible version). The console shows the error below this line. |

ItemsAdder, Nexo, Oraxen and CraftEngine load their content after the server starts. Until then the line ends with
`waiting for data`, and a second line `<name> data loaded` (e.g. `itemsadder data loaded`) appears when the content
is ready.

!!! note
    Door+ looks for the plugins only when it starts. If you install, remove or update one of these plugins, restart
    the server. `/dp admin reload` does not connect or disconnect integrations.

## Switching an integration off

Every integration has a switch in `config.yml`. Set it to `false` to keep Door+ away from a plugin even when that
plugin is installed:

```yaml
integrations:                  # false = do not hook even if the plugin is installed
  luckperms: true
  placeholderapi: true
  worldguard: true
  itemsadder: true
  nexo: true
  oraxen: true
  craftengine: true
  citizens: true
  fancynpcs: true
```

Restart the server after changing a switch. See [Configuration](../reference/configuration.md).

## Seeing the active integrations

Run `/dp admin version` (permission `doorplus.admin.version`). Two lines matter here:

| Line | Shows |
|---|---|
| **Integrations:** | The integrations that started, by their `config.yml` name, e.g. `luckperms, worldguard, itemsadder, placeholderapi`, or `none`. |
| **Hooks and providers:** | What is plugged into Door+ right now: `protection:<name>` (protection checks such as `protection:WorldGuard`), `content:<name>` (custom block plugins such as `content:itemsadder`), `npc:<name>` (NPC plugins such as `npc:fancynpcs`), or `none`. Hooks added by other plugins through the API appear here too. |
