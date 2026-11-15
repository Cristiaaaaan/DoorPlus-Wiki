# Language

This page explains how Door+ chooses the language of its messages and menus, how to edit the texts, and how to add a new language. It is for server administrators.

## Language files

- The files are in `plugins/DoorPlus/lang/`. Door+ ships with `en.yml` (English) and `it.yml` (Italian).
- On every start and every `/dp admin reload`, Door+ copies a bundled file into that folder **only if it is missing**. It never overwrites a file that is already there, so your edits are safe across updates.
- Every `.yml` file in the folder is loaded. The file name without `.yml` is the language code: `de.yml` is the language `de`.
- Every chat message, menu title, button name, item lore and chat prompt of Door+ comes from these files.

## Choosing the language

Two keys in `config.yml` control it (see [Configuration](configuration.md#language)):

| Key | Default | Effect |
|---|---|---|
| `language.default` | `en` | Language for the console, and for players when `per-player` does not apply. If no file with that name exists, Door+ writes a warning and uses `en`. |
| `language.per-player` | `true` | Each player sees the language of their Minecraft client, if a matching file exists. |

With `per-player: true`, Door+ takes the language part of the client setting: a client set to `de_de` (Deutsch) uses `de.yml`, a client set to `pt_br` uses `pt.yml`. If that file does not exist, the player gets `language.default`.

To show one language to everybody, set `per-player: false` and choose `language.default`.

## Editing messages

1. Open `plugins/DoorPlus/lang/en.yml` (or the language you want to change).
2. Change the text after the key. Keep the key and the indentation as they are.
3. Save the file.
4. Run `/dp admin reload`. You see **Configuration and language files reloaded.**

Example: change the "door not found" message.

```yaml
general:
  door-not-found: '<red>No such door. Check the name with <white>/dp list</white>.'
```

!!! tip "Keep the quotes"
    Put every text in single quotes. To write a single quote inside the text, double it: `'You can''t do this.'`

### Formatting with MiniMessage

Texts use the [MiniMessage](https://docs.advntr.dev/minimessage/format.html) format:

| You write | Result |
|---|---|
| `<red>Text` | red text |
| `<gold>Door+</gold> rest` | "Door+" in gold, the rest unchanged |
| `<b>bold</b>`, `<i>italic</i>`, `<u>underlined</u>` | text styles |
| `<#ff8800>Text` | any hex colour |
| `<gradient:gold:red>Text</gradient>` | colour gradient |

- To show a literal `<` that is not a tag, write `\<`, as the bundled files do in the command syntax lines (`'/door info \<door>'`).
- Menu item names and lore are not italic unless you add `<i>`.
- Some keys are lists, one entry per line (for example `command.info.body` or item lore). Keep them as lists.
- `prefix` is put in front of most chat messages. Set it to `''` to remove it everywhere.

### Hiding a message

Set a message to an empty text to stop Door+ from sending it:

```yaml
command:
  toggle:
    success: ''
```

### Placeholders

Many messages contain placeholders in angle brackets, such as `<door>`, `<player>`, `<count>`, `<limit>` or `<id>`. Door+ replaces them with the real value when it sends the message.

- You can move a placeholder, remove it, or style it: `'<green>Door <white><door></white> locked.'`
- Only the placeholders that the original text of that key uses are available in it. A placeholder copied from another key is not replaced.
- Values are inserted as plain text: a door or player name can never add colours or formatting.

PlaceholderAPI placeholders (`%...%`) are not supported inside Door+ messages. Door+'s own PlaceholderAPI placeholders are listed on the [Placeholders](placeholders.md) page.

## Adding a new language

1. Copy `plugins/DoorPlus/lang/en.yml` to a new file named with the two-letter language code, for example `de.yml` for German or `fr.yml` for French.
2. Translate the texts. Do not translate the keys.
3. Run `/dp admin reload`.
4. Players whose client is set to that language now see it (if `language.per-player` is `true`).
5. Optional: to make it the language of the console and of everyone else, set `language.default: de` in `config.yml` and run `/dp admin reload` again.

!!! note
    The file name must match the client language code (`de`, `fr`, `es`, `pt`, ...), in lower case, not the full locale (`de_de`).

## After an update

New Door+ versions can add new message keys. Your edited files are not overwritten, so they do not contain the new keys yet. Door+ handles this for you:

- If a key is missing from your file, Door+ uses the text from the bundled file of the same language, or the bundled English text if the language is not bundled (for example your own `de.yml`).
- The first time a missing key is used, the console shows a warning such as `Missing language key 'admin.claim.success' in lang/de.yml; using the bundled text`.

To bring the new keys into your file:

1. Read the key names from the console warnings.
2. Copy those keys with their text from the bundled file into your file and translate them. To see the bundled file of a bundled language, rename your file (for example to `en-custom.yml.old`), run `/dp admin reload`, and Door+ writes a fresh `en.yml`.
3. Run `/dp admin reload`.

!!! warning "Resetting a file"
    Deleting `en.yml` or `it.yml` and running `/dp admin reload` restores the original texts, and removes all your edits in that file.
