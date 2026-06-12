# RuneLite Rules And Compliance

Use this reference before implementing or reviewing features involving combat, PvP, menus, interfaces, input, player data, HTTP, persistence, external processes, runtime loading, or other Plugin Hub risk areas.

Sources:

- RuneLite example-plugin AGENTS.md: https://github.com/runelite/example-plugin/blob/master/AGENTS.md
- RuneLite Rejected or Rolled Back Features: https://github.com/runelite/runelite/wiki/Rejected-or-Rolled-Back-Features
- Jagex Third-Party Client Guidelines: https://secure.runescape.com/m=news/third-party-client-guidelines?oldschool=1
- RuneLite Plugin Hub README: https://github.com/runelite/plugin-hub

## Hard Bans

- Keep hub plugins Java-only and Java 11 compatible.
- Do not use reflection.
- Do not use JNI, JNA, direct native memory access, `Unsafe`, or LWJGL native access.
- Do not execute external programs with `Process`, `ProcessBuilder`, shell commands, or equivalents.
- Do not download, vendor, or dynamically load code at runtime, including classloading.
- Do not generate code at runtime.
- Do not use Java serialization.
- Do not automate RuneScape gameplay, inject mouse/keyboard input, or use computer-use tools to interact with RuneScape for verification.

## Boss And Combat Restrictions

Apply these conservatively to bosses, raid sub-bosses, Slayer bosses, demi-bosses, Fight Caves, Inferno, and other wave-based minigames:

- Do not predict next attacks by timing, style, counter, animation, or future state.
- Do not show projectile targets or landing indicators.
- Do not recommend or indicate prayer switches.
- Do not show attack counters or flinch timers.
- Do not automatically indicate where to stand or avoid standing. Manual tile marking is acceptable.
- Do not add visual or audio warnings for future boss mechanics.
- Highlight only current, already-active hazards when that is otherwise allowed.
- Do not identify NPC focus or which player an NPC is targeting.
- Do not simulate boss fights or other content.
- Treat new high-end PvM boss plugins as rejected by default.

## PvP Restrictions

- Do not remove, hide, deprioritize, or simplify attack/cast options in PvP.
- Do not show opponent freeze duration indicators.
- Do not identify PvP clan opponents or a player's opponent.
- Do not preview PvP loot drops.
- Do not provide PvP target scouting information.
- Do not summarize attackable players, prayer use, gear, or similar player group information.
- Do not highlight players based on PvP level range or attackability.
- Do not simplify spell targeting by removing menu options.

## Menu, Interface, And Input Restrictions

- Do not add menu entries that send game actions to the server.
- Do not modify menus for Construction or Blackjacking.
- Avoid conditional menu entry removal based on NPC type, friend status, or similar power conditions.
- Do not unhide hidden interface components such as special attack bar or minimap.
- Do not move or resize 3D click zones, combat options, inventory, equipment, prayer book, or spellbook click zones/components.
- Do not remove the inventory pane background or make it click-through.
- Do not allow detached-camera world interaction.
- Do not inject mouse or keyboard events.
- Do not programmatically insert text into the chatbox, including paste or shorthand expansion.
- Do not modify outgoing chat messages after the user sends them.

## Data, Privacy, And Content Restrictions

- Do not expose player information over HTTP.
- Do not crowdsource data about other players, including locations, gear, names, or identity mappings.
- Do not create credential-manager plugins that store account credentials.
- Do not implement adult or overtly sexual content.
- Do not rely on player-provided IDs for an entire plugin's functionality.

## Logging, Threading, And Performance

- Use `log.debug()` for diagnostics.
- Do not use `log.info()` for per-frame or high-frequency event logging; RuneLite runs at INFO in production.
- Never use `Thread.sleep()`.
- Do not block in `startUp()` or `shutDown()`.
- On shutdown, cancel scheduled tasks explicitly and call `shutdownNow()`; do not wait indefinitely.
- Never perform blocking network or disk I/O on the client thread.
- Use OkHttp `enqueue()` for network work; call back into `client` through `clientThread.invoke()`.
- Use `CompletableFuture.allOf()` for batched async work instead of `CountDownLatch`.
- If a timeout-capable wait is unavoidable, use a reasonable timeout.
- Do not scan the whole scene every tick or frame. Track relevant objects/NPCs through spawn/despawn and other events.
- Keep overlay computation minimal.

## HTTP, JSON, File I/O, And Config

- Use injected `OkHttpClient` for HTTP. Do not use `HttpURLConnection`, `java.net.http.HttpClient`, or Apache HttpClient.
- Use injected `Gson`; derive variants with `gson.newBuilder()` when needed.
- Do not add transitive RuneLite client dependencies such as Gson, Guice, or OkHttp directly to `build.gradle`.
- Store plugin-owned files only under `.runelite`, using `RuneLite.RUNELITE_DIR`, unless the user explicitly selects a path through `JFileChooser`.
- Use specific config group names.
- Never rename config groups or keys without a migration.
- Any config item that toggles a third-party server feature must be disabled by default and include this warning exactly: `This feature submits your IP address to a 3rd-party server not controlled or verified by RuneLite developers`

## Packaging And Assets

- Rename all template identifiers: package, plugin class, config class, config group, Gradle group, settings project name, and `runelite-plugin.properties`.
- Do not include `META-INF/services/net.runelite.client.plugins.Plugin`.
- Do not commit build artifacts such as `.class` files, `out/`, or `.tmp/`.
- Keep `build.gradle` Java 11 compatible and aligned with the RuneLite example-plugin structure.
- Retain a permissive license, such as BSD-2.
- Optimize icon PNG dimensions and verify image formats are truly PNG when named `.png`.
