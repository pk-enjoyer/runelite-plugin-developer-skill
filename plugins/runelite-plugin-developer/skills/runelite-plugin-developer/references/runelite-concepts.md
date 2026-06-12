# OSRS And RuneLite Concepts

Use this reference when reasoning about what Old School RuneScape is, what RuneLite is, how OSRS state reaches a RuneLite plugin, what users can do through RuneLite, and where plugin boundaries are.

Sources:

- OSRS Wiki: https://oldschool.runescape.wiki/
- OSRS Wiki Game tick: https://oldschool.runescape.wiki/w/Game_tick
- OSRS Wiki Java: https://oldschool.runescape.wiki/w/Java
- OSRS Wiki RuneScript: https://oldschool.runescape.wiki/w/RuneScript
- OSRS Wiki RuneLite: https://oldschool.runescape.wiki/w/RuneLite
- RuneLite Developer Guide: https://github.com/runelite/runelite/wiki/Developer-Guide
- RuneLite Plugin Hub README: https://github.com/runelite/plugin-hub
- RuneLite API Javadocs: https://static.runelite.net/runelite-api/apidocs/index.html
- RuneLite Rejected or Rolled Back Features: https://github.com/runelite/runelite/wiki/Rejected-or-Rolled-Back-Features
- Jagex Third-Party Client Guidelines: https://secure.runescape.com/m=news/third-party-client-guidelines?oldschool=1

## Mental Model

- Old School RuneScape is a server-authoritative online game. Jagex servers decide actual game outcomes: movement, combat, inventory changes, object interactions, chat delivery, world membership, deaths, drops, and state transitions.
- RuneLite is a local Java client that renders the game, receives server updates, manages local UI/client state, and exposes a public API for plugins to observe and augment the client.
- A RuneLite plugin runs locally inside the RuneLite process. It can read exposed client state, subscribe to events, render overlays, add sidebar panels, maintain local state, and use allowed client services.
- A plugin is not a game server extension. It should not assume authority over game state, bypass server rules, automate actions, or reveal future state that the client has not legitimately received.
- The user remains the actor. Acceptable plugin design generally informs, annotates, records, filters local views, or makes local UI more understandable; it does not play the game for the user.
- OSRS Wiki is a high-value source for game mechanics, terminology, content behavior, item/NPC/object names, player-facing explanations, and context that RuneLite Javadocs intentionally do not provide. Use it to understand the game before choosing APIs.

## State Flow

1. Jagex servers send updates to the OSRS client.
2. The RuneLite client processes those updates into local state such as actors, tiles, widgets, item containers, vars, chat messages, menu entries, and cache definitions.
3. RuneLite posts API/client events as local state changes or client lifecycle points occur.
4. Plugins receive events through `@Subscribe`, read current state through `Client` and injected services, update plugin-owned models, and render local UI.
5. The user sees RuneLite's rendered game plus plugin overlays/panels and chooses any real game actions through normal game input.

Treat this as an observation pipeline. Plugin state can lag, be incomplete, or reset across login, hop, region load, widget reload, or plugin shutdown. Always handle `null`, hidden widgets, unloaded regions, despawned actors, and non-logged-in `GameState`.

## Ticks And Timing

- OSRS game ticks are the server cycle and fundamental unit for server-processed actions, normally 0.6 seconds. `GameTick` fires after game packets for that tick have been processed.
- Server actions registered during one tick generally begin on the next tick, so apparent delay can range from nearly instant to nearly one full tick.
- Client-only UI actions, such as opening a right-click menu or switching interface tabs, can be processed separately from server ticks. `ClientTick` is much more frequent and should not be used as a substitute for server game-tick logic unless the feature is truly local UI behavior.
- Treat wall-clock tick timing as approximate. Server lag, client lag, connection quality, and world population can make observed ticks longer or cause missed ticks.
- Rendering is frame-based and can happen often. Keep overlays cheap and render only from precomputed state when possible.
- Avoid predicting future combat, PvP, boss, projectile, or hazard state. If a feature needs timing logic, check Plugin Hub and rejected-feature rules before building it.

## OSRS Wiki Research

- Search OSRS Wiki when a task mentions a boss, minigame, interface, item, NPC, object, spell, var-like mechanic, tick timing, RuneScript behavior, or player-facing workflow that is not obvious from the code.
- Use OSRS Wiki to learn terminology and mechanics, then verify implementation details against RuneLite APIs, local code, gameval constants, and Plugin Hub rules.
- Do not treat OSRS Wiki as a substitute for RuneLite API docs or compliance rules. It explains game behavior; it does not define what Plugin Hub will approve.
- Prefer stable game concepts from the wiki over guesses, but keep plugin logic resilient because OSRS content and wiki pages can change.

## Java And RuneScript Context

- RuneScape's game engine is mostly Java. RuneLite plugins are also Java, but they are separate client-side code running inside RuneLite, not Jagex engine code.
- RuneScript is Jagex's content scripting language for OSRS content. The engine is not written in RuneScript; RuneScript content is translated before the engine can run it.
- RuneScript explains why many game actions are trigger-like: button clicks, item options, NPC/object operations, client scripts, and content definitions can appear to plugins as widgets, menu options, vars, cache definitions, and client script effects.
- Plugins should observe RuneScript-driven results through public RuneLite APIs. Do not assume access to source RuneScript, do not reproduce server/client scripts through reflection, and do not send actions just because a RuneScript trigger exists.
- Java data limits can explain OSRS values such as 32-bit item stack limits, 16-bit-style counters, and XP caps. Use appropriate numeric types in plugin code: `long` for computed GP totals or accumulated values even when individual game quantities are `int`.

## Surfaces A Plugin Can Observe

- Actors: local player, players, NPCs, animations, graphics, health bars, interacting targets, overhead text.
- Scene: world views, tiles, game objects, ground objects, wall objects, decorative objects, ground items, projectiles, graphics objects.
- UI: widgets, widget items, interface loads/closes, menu entries, menu sorting, chatbox messages, sidebar panels, overlays.
- Inventory-like state: item containers for inventory, equipment, bank-like interfaces, loot containers, and other exposed containers.
- Vars and settings: varbits, varplayers, varclients, game state, world, account/client profile context where available.
- Cache data: item/object/NPC definitions, enums, structs, DB tables, sprites, params, and composition post-processing events.

Prefer the API surface closest to the real state. For example, use `ItemContainer` for item IDs/quantities and widgets for layout/bounds; use vars for game state flags instead of parsing text; use spawn/despawn events instead of scanning all scene tiles.

## User Interaction Boundaries

- Reading menu entries is often useful for context, hover state, and overlays. Creating or modifying entries is high risk because menu entries can represent server actions.
- Drawing overlays or panels is local UI augmentation. It does not change OSRS state by itself.
- Programmatically clicking, typing, selecting spells/items, injecting input, changing outgoing chat, or sending game actions crosses into automation or server-action territory and is generally disallowed for Plugin Hub work.
- Do not make hidden game UI visible, move protected click zones, make panes click-through, or simplify PvP/combat targeting in ways that affect gameplay actions.
- Manual user configuration is acceptable when it stores local preferences. Keep config keys stable and avoid silently changing existing user state.

## Client Thread And Threads

- Treat `Client` and most `net.runelite.api` objects as client-thread state. Read or mutate them from event handlers or through `ClientThread.invoke()` when crossing thread boundaries.
- Network callbacks, executor tasks, Swing listeners, and tests can run off the client thread. Copy plain data out of client state before doing background work.
- Swing UI updates belong on the EDT. Keep Swing panels passive and let services/controllers own state changes.
- Never block startup, shutdown, the client thread, event handlers, or render paths with network, disk, sleep, or long computations.

## Login, Worlds, Regions, And Resets

- Plugins must handle startup before login, logout, world hop, disconnect, region reload, instanced areas, and plugin disable/enable.
- Clear or revalidate actor, tile, widget, menu, and container references on `GameStateChanged`, relevant despawn/unload events, and `shutDown()`.
- Player names are not durable identity. Prefer RuneLite profile/account APIs or config scoping where available. Be conservative with any player data, especially if HTTP or persistence is involved.
- Instanced regions and multiple world views can make simple world-coordinate assumptions wrong. Use `WorldView`, `WorldPoint`, `LocalPoint`, and `Perspective` deliberately.

## Design Heuristics

- Start with the user's real workflow: what they see, decide, click, configure, or review.
- Decide whether the feature is informational, visual, persistence-oriented, networked, or action-affecting. Action-affecting features require the strictest compliance review.
- Ask "what is the authoritative source?" before implementing: server-updated API state, local widget state, a var, cache definition, user config, or plugin-owned history.
- Prefer robust state models over UI scraping. UI text and child indexes are brittle unless the interface itself is the only available source.
- Make failure quiet and reversible: hide overlays when state is unavailable, clear stale state on unload, and avoid noisy logs.
- Validate with automated tests for pure logic and manual in-game smoke tests for behavior that depends on live client state.
