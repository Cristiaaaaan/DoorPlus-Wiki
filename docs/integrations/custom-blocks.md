# Custom blocks and furniture

This page explains how to use the custom blocks, furniture and items of ItemsAdder, Nexo, Oraxen and CraftEngine with
Door+. It is for server owners and door owners on servers with one of these plugins.

With one of these plugins installed, Door+ adds:

| Feature | Trigger type | Page |
|---|---|---|
| A placed custom block opens the door when right-clicked | **Custom block** | [Triggers](../access/triggers.md) |
| A placed furniture opens the door when right-clicked | **Furniture** | [Triggers](../access/triggers.md) |
| A custom item works as the door key | — | [Keys](../access/keys.md) |

No configuration is needed. Each plugin can be switched off with `integrations.itemsadder`, `integrations.nexo`,
`integrations.oraxen` or `integrations.craftengine` in `config.yml` (see [Integrations](index.md)).

## Wait until the content is loaded

These plugins load their blocks and items a little after the server starts. Door+ recognises custom content only
after that. The console shows `<plugin> data loaded`, e.g. `itemsadder data loaded`.

!!! warning
    If you link a custom block before its plugin has loaded, Door+ sees an ordinary block and creates a normal
    **Block** trigger. If you click furniture too early you see **This object is not recognised yet. Try again in a
    moment.** Wait for the console line, then link again.

## Linking a custom block

You need the **Admin** role on the door and the permissions `doorplus.trigger.block` and
`doorplus.trigger.custom_block` (every player has them by default through `doorplus.user`).

=== "Menu"

    1. Open the door menu and click **Triggers**.
    2. Click **Add trigger**, then **Block**.
    3. Click the custom block in the world.

=== "Command"

    1. Run `/dp trigger add <door> block [action]`, e.g. `/dp trigger add castle_gate block toggle`.
    2. Click the custom block in the world.

You see **Linked Custom block trigger #&lt;id&gt; to &lt;door&gt;.** If Door+ says **Block** instead of
**Custom block**, the plugin had not loaded yet (see above).

## Linking furniture

You need the **Admin** role on the door and the permissions `doorplus.trigger.entity` and
`doorplus.trigger.furniture` (every player has them by default).

=== "Menu"

    1. Open the door menu and click **Triggers**.
    2. Click **Add trigger**, then **Entity**.
    3. Click the furniture in the world.

=== "Command"

    1. Run `/dp trigger add <door> entity [action]`, e.g. `/dp trigger add castle_gate entity open`.
    2. Click the furniture in the world.

You see **Linked Furniture trigger #&lt;id&gt; to &lt;door&gt;.**

!!! note "ItemsAdder complex furniture"
    Complex (multi-entity) furniture works like simple furniture. Door+ stores the id of its custom entity, which
    can differ from the id of the furniture item: `/dp trigger inspect` shows the stored id.

!!! note "Oraxen shaped blocks"
    Custom stairs, slabs, doors and trapdoors of Oraxen work as custom blocks. A two-block door counts only on the
    half you linked.

## How these triggers behave

- A trigger fires when a player **right-clicks** the custom block or furniture.
- Available actions: **Toggle**, **Open**, **Close**, and **Go to stop** for platforms. **Follow state** is not
  available, because a custom block has no on/off state.
- The player must be allowed to use the door, as for any trigger (see [Triggers](../access/triggers.md)).
- By default the click is consumed: the plugin's own action for that block or furniture (for example sitting on a
  chair) does not happen. To let it happen as well, run `/dp trigger set <trigger id> cancel-vanilla false`.

## Custom item ids

Door+ stores the id each plugin gives to its content:

| Plugin | Id format | Example |
|---|---|---|
| ItemsAdder | `namespace:id` | `castle:gate_lever` |
| Nexo | item id | `gate_lever` |
| Oraxen | item id | `gate_lever` |
| CraftEngine | `namespace:id` | `castle:gate_lever` |

The provider column of the trigger menu and of `/dp trigger list` shows the plugin (`itemsadder`, `nexo`, `oraxen`,
`craftengine`).

## Custom items as keys

Any custom item from these plugins can be the key of a door. All items with the same custom id open the door, so
you can sell or craft keys with your item plugin.

=== "Menu"

    1. Hold the custom item in your main hand.
    2. Open the door menu, click **Key**, then **Use the custom item in hand**.
    3. Set the **Mode** to **Key as alternative** or **Key required**.

=== "Command"

    1. Hold the custom item in your main hand.
    2. Run `/dp key custom <door>`. You see **The custom item in your hand is now the key of &lt;door&gt;.**
    3. Run `/dp key mode <door> alternative` or `/dp key mode <door> required`.

The key is saved as `CUSTOM:<plugin>:<id>`, e.g. `CUSTOM:itemsadder:castle:gate_key`. Ids are compared without
regard to upper and lower case. If the item in your hand is not a custom item, you see **The item in your hand is not
a custom item recognised by Door+.**

Modes, consumption and where the key must be held are explained in [Keys](../access/keys.md).

## When the source or item disappears

| What happens | Result |
|---|---|
| Someone breaks the linked custom block or furniture | The trigger is removed. If the door owner is online (and is not the one who broke it), they get **The source of trigger #&lt;id&gt; (&lt;type&gt;) of &lt;door&gt; at x y z disappeared: the trigger was removed.** |
| The door has **Protect blocks** on | Only players with the Admin role (or a bypass permission) can break its trigger sources. Others see **This object is a protected trigger source.** |
| A different custom block now stands where the linked one was | The trigger is removed the next time someone clicks it. |
| The plugin replaces the furniture with a new copy of the same item at the same block | The trigger moves to the new copy automatically the first time someone clicks it. |
| The block or furniture vanishes without being broken (removed by a command, a world edit, or the item is deleted from the plugin's configuration) | The trigger stays in the list but no longer fires. Remove it from the trigger menu or with `/dp trigger remove <trigger id>`. |
| The custom content plugin is uninstalled | Its triggers stay saved but do nothing. They work again if you reinstall the plugin. |
| The key item is deleted from the plugin, or the plugin is uninstalled | No item matches the key any more. With **Key required**, only the owner, Admins and players with `doorplus.admin.bypass.use` can use the door. Set a new key or remove it with `/dp key remove <door>`. |
