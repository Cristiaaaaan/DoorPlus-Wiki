# Keys

A key is an item that lets players use a door. This page explains the three key modes, how to create a
key from any item or from a custom item, how to give copies, and how to make old keys stop working. It is
for door owners and door admins.

## Key modes

| Mode | Command value | Menu name | Effect |
|---|---|---|---|
| None | `none` | **No key** | The door has no key. Only privacy and access entries count. |
| Alternative | `alternative` | **Key as alternative** | Holding the key is enough, even for players who are not allowed by the privacy or access list. |
| Required | `required` | **Key required** | Players must be allowed by the privacy or access list **and** hold the key. |

- The owner and the admins of the door never need the key, in any mode. Neither do players with
  `doorplus.admin.bypass.use`.
- A trigger with access mode **Public** ignores the key. With the other access modes the key mode applies
  as usual. See [How a use is decided](privacy-and-roles.md#how-a-use-is-decided).
- The key is checked on every use: triggers, proximity zones, `/dp toggle` and the menu. Players without a
  role on the door cannot find it with commands or menus, so key holders without a role use the door
  through its triggers.
- A locked door does not open, key or not.

You can only choose **Key as alternative** or **Key required** after the door has a key. Otherwise:
**This door has no Door+ key yet. Create one first.**

## Holding the key

`keys.check` in the config decides where the key must be:

| `keys.check` | The player has the key when |
|---|---|
| `HAND` (default) | The key is in the main hand or the off hand. |
| `INVENTORY` | The key is anywhere in the inventory. |

## Who can manage keys

The owner and the admins of the door (role **Admin** or higher). By command you also need
`doorplus.command.manage`, which every player has by default.

## Open the key menu

1. Open the door menu with `/dp menu <door>`.
2. Click **Key**. The menu **Key — `<door>`** opens.

The first item, **Key of `<door>`**, shows the mode, the current key (**none** if there is no key) and
whether the key is consumed on use.

## Create a key from the item in your hand

Any item can become a key. Door+ marks the item, so a renamed or copied item from another source does not
work: only items marked by Door+ for this door do.

=== "Menu"

    1. Hold the item in your main hand. You can hold a whole stack: every item of the stack becomes a key.
    2. In **Key — `<door>`**, click **Create from the item in hand**.
    3. Chat confirms: **The item in your hand is now a key of `<door>`.**
    4. Click **Mode** to choose **Key as alternative** or **Key required** (left click: next, right click:
       previous).

=== "Command"

    1. Hold the item in your main hand.
    2. Run:
       ```
       /dp key create <door>
       ```
    3. Choose the mode:
       ```
       /dp key mode <door> <none|alternative|required>
       ```

Details:

- The first time, Door+ generates a key id for the door. Later uses mark more items with the **same** id,
  so every key made this way opens the door.
- One item can be the key of up to 16 doors. Beyond that: **This item already opens too many doors.**
- With an empty hand: **Hold an item in your main hand.**

## Use a custom item as key

With ItemsAdder, Nexo, Oraxen or CraftEngine installed, you can make every item with a given custom id a
key, for example a keycard. Items do not need to be marked by Door+.

=== "Menu"

    1. Hold the custom item in your main hand.
    2. In **Key — `<door>`**, click **Use the custom item in hand**.
    3. Chat confirms: **The custom item in your hand is now the key of `<door>`.**
    4. Choose the **Mode**.

=== "Command"

    ```
    /dp key custom <door>
    ```

If the item is not a recognised custom item: **The item in your hand is not a custom item recognised by
Door+.** See [Custom blocks and furniture](../integrations/custom-blocks.md).

!!! note
    A door has one key at a time. Using a custom item replaces a Door+ key, and creating a key from the
    item in hand replaces a custom key. **Give a copy** and **Regenerate** work only with a Door+ key, not
    with a custom item key.

## Give copies

=== "Menu"

    In **Key — `<door>`**, click **Give a copy**. You get a new key item.

=== "Command"

    ```
    /dp key give <door> [player]
    ```

    Without `[player]` the copy goes to you. The target player must be online.

The copy is a `keys.default-item` (default `TRIPWIRE_HOOK`) named **Key of `<door>`**, with the door name
and the owner in its description. If the inventory is full, the item drops at the player's feet.

## Consume on use

When **Consume on use** is on, one key item is removed each time the key was what let the player in and
the door moves. Owners and admins never lose keys, and players who were allowed without the key (for
example on a `PUBLIC` door with **Key as alternative**) do not lose them either.

=== "Menu"

    In **Key — `<door>`**, click **Consume on use** to toggle it.

=== "Command"

    ```
    /dp key consume <door> <true|false>
    ```

!!! tip
    Use **Consume on use** with **Give a copy** to sell single-use tickets, for example for an arena gate.

## Make old keys stop working

**Regenerate** gives the door a new key id. Every Door+ key already given out stops working at once. Make
new keys afterwards with **Create from the item in hand** or **Give a copy**.

=== "Menu"

    1. In **Key — `<door>`**, click **Regenerate**.
    2. Confirm. Chat shows: **New key for `<door>`: old keys no longer work.**

=== "Command"

    ```
    /dp key reset <door>
    ```

## Remove the key

Removes the key and sets the mode back to **No key**.

=== "Menu"

    In **Key — `<door>`**, click **Remove the key**.

=== "Command"

    ```
    /dp key remove <door>
    ```

## Config keys

| Key | Default | Meaning |
|---|---|---|
| `keys.check` | `HAND` | `HAND`: main or off hand. `INVENTORY`: anywhere in the inventory. |
| `keys.default-item` | `TRIPWIRE_HOOK` | Material of the copies made by **Give a copy**. |

See [Configuration](../reference/configuration.md).
