# Licence, verification and privacy

This page is for server owners. It explains how Door+ verifies your purchase, what it sends over the internet,
what happens if the check fails, the anonymous statistics it collects, and a summary of the licence terms.

## Where Door+ is sold

Door+ is sold only on **BuiltByBit** and **Voxel Shop**. When you download the jar, the marketplace writes a
download identifier (and your buyer name) into it. Door+ uses that identifier to check the purchase with the same
marketplace. There is no Door+ licence server, no account to create, no IP lock and no limit on the number of your
own servers.

## How the purchase check works

1. **At start-up**, Door+ checks that the jar contains a BuiltByBit or Voxel Shop download identifier. If it does
   not (the jar was not downloaded from a marketplace, or was rebuilt), Door+ disables itself immediately and logs:

    ```
    This copy of Door+ was not downloaded from BuiltByBit or Voxel Shop. Door+ only runs from a purchased download; disabling.
    ```

2. If the jar contains your buyer name, the console shows `Licensed to <name> (<id>) via BuiltByBit` (or
   `via Voxel Shop`) followed by the resource and download ids. `/dp admin version` shows the same line as
   **Licensed to:**.
3. About one second after start-up, and then **every 12 hours**, Door+ asks the marketplace licence API whether the
   download is still licensed. The request runs in the background and never slows down the server.
4. Door+ acts on the answer:

| Answer | What Door+ does |
|---|---|
| Licensed | Keeps running and records the successful check |
| No usage rights (refunded, revoked or shared copy) | Disables itself (see [below](#if-the-purchase-is-refused)) |
| No answer (network error, timeout, rate limit, marketplace error) | Keeps running, logs a warning once and tries again 12 hours later. Disables itself only after 7 days without any successful check |

Only an explicit "no usage rights" answer from the marketplace disables Door+ at once. A problem on the
marketplace's side or on your network never does.

### What is sent

| Marketplace | Request | Data sent |
|---|---|---|
| BuiltByBit | `GET https://api.builtbybit.com/v2/resources/embed/latest` | The download nonce written into your jar, Door+'s own API token, and a `User-Agent` of `DoorPlus/<version>` |
| Voxel Shop | `POST https://api.voxel.shop/v1/verifyPurchase` | The licence key and resource id written into your jar, and a `User-Agent` of `DoorPlus/<version>` |

Nothing about your server, worlds, players or configuration is sent. As with any internet request, the marketplace
sees the IP address of your server.

### The 7-day grace period

If the marketplace cannot be reached, Door+ logs:

```
Could not verify the Door+ purchase right now; retrying later. Door+ keeps working for up to 7 days without a successful check.
```

Door+ keeps working for 7 days counted from the first failed check since the last successful one. One successful
check resets the count. If the 7 days pass with no successful check, Door+ logs the following and disables itself:

```
The purchase of Door+ could not be verified for 7 days (marketplace unreachable); disabling. Check that the server can reach the internet.
```

The state of the grace period is saved in `plugins/DoorPlus/.license`, signed for your download. A copy of this file
taken from another server, or edited by hand, is ignored.

!!! tip
    If your host blocks outgoing connections, allow HTTPS to `api.builtbybit.com` (BuiltByBit purchases) or
    `api.voxel.shop` (Voxel Shop purchases).

## If the purchase is refused

When the marketplace reports that the download has no usage rights, Door+ logs:

```
The marketplace reports that this download of Door+ has no usage rights (refunded, revoked or shared copy); disabling.
```

Then **only Door+** is disabled, exactly as in a normal shutdown:

- New movements are refused, and running animations are finished at once: every door is left in the world as
  normal blocks, in its final position.
- Windmills, flags and clocks are stopped and their blocks are put back in the world.
- Open menus, door creations and chat prompts are closed, and pending database writes are saved.
- The Door+ API is no longer available to other plugins.
- Your worlds, other plugins and the rest of the server are not changed in any way.

After that, `/dp` commands answer **Door+ is disabled. Details are in the server console.**

### If Door+ was disabled by mistake

1. Read the console line that explains why (one of the messages above).
2. "not downloaded from BuiltByBit or Voxel Shop": download the jar again from your purchases page on the
   marketplace and use it as it is. Do not rename the classes, repack or re-sign the jar.
3. "could not be verified for 7 days": check that the server can reach the marketplace address listed in the tip
   above, then restart the server. The check runs again at start-up.
4. "has no usage rights": the marketplace considers the purchase refunded or revoked. Contact the author through
   the marketplace page or the [support Discord](https://discord.gg/bHVeABaWF) with your marketplace username and
   the console log. Do not share your jar with anybody: a shared download can be revoked.

## Obfuscation

Release jars are obfuscated: class and method names are renamed. Nothing is removed or optimised, so the plugin
behaves exactly like the unobfuscated build. The public API (`io.github.cristiaaaaan.doorplus.api`) and the main
plugin class keep their names, so other plugins can use them normally. Stack traces in the console look scrambled;
send them unchanged in bug reports (see [Troubleshooting](troubleshooting.md#collecting-information-for-a-bug-report)).

## Anonymous statistics (bStats)

Door+ sends anonymous usage statistics to [bStats](https://bstats.org/plugin/bukkit/DoorPlus/35344) (plugin id
`35344`). Besides the general data bStats collects for every plugin, Door+ adds these charts:

| Chart | Content |
|---|---|
| `doors` | Number of doors on the server |
| `triggers` | Number of triggers |
| `door_types` | Number of doors of each type |
| `storage` | Storage type: `sqlite`, `mysql` or `mariadb` |
| `animation_engine` | `block_display` or `falling_block` |
| `integrations` | Active integrations (`luckperms`, `worldguard`, `placeholderapi`, ...) or `none` |

No door names, coordinates, player names or configuration values are sent.

**To turn statistics off:**

1. Open `plugins/bStats/config.yml`.
2. Set `enabled: false`.
3. Restart the server.

This turns bStats off for every plugin that uses it, not only Door+.

## Licence terms (summary)

The full text is the end user licence agreement (EULA) included with Door+. In short:

| Topic | Terms |
|---|---|
| What you get | A non-exclusive, non-transferable licence to install and run Door+ on Minecraft servers that you own or administer |
| Not allowed | Redistributing, reselling, sharing, sublicensing or uploading the plugin (or a modified version), free or paid; removing or changing the buyer identification written by the marketplace |
| Allowed changes | Configuration and language files, for your own servers. Decompiling or modifying the code only for personal use on your own servers, never for distribution |
| Public API | The `DoorPlus-API` artifact is under the MIT licence, not under the EULA |
| Purchase verification | Described on this page; the plugin never alters or damages the server or its worlds |
| Updates and support | At the author's discretion, through the marketplace page |
| Warranty | None: the plugin is provided "as is" |
| Termination | Any breach ends the licence; the plugin must then be removed from every server |
