# RuneLite Plugin Patterns

Use this reference when changing plugin lifecycle, dependency injection, config, overlays, Swing panels, event subscribers, HTTP/JSON/file I/O, or resource loading. Read [runelite-concepts.md](runelite-concepts.md) first when the change depends on OSRS/RuneLite/user interaction boundaries or OSRS mechanics that should be researched on OSRS Wiki. Read [java-quality.md](java-quality.md) when implementation quality, concurrency, logging, hot-path performance, resource handling, or tests are in scope.

Sources:

- RuneLite Developer Guide: https://github.com/runelite/runelite/wiki/Developer-Guide
- RuneLite Plugin Hub README: https://github.com/runelite/plugin-hub
- RuneLite example-plugin AGENTS.md: https://github.com/runelite/example-plugin/blob/master/AGENTS.md

## Repository Inspection

- Start with `git status --short`; avoid reverting unrelated local work.
- Read `settings.gradle`, `build.gradle`, `runelite-plugin.properties`, the main plugin class, config interface, overlays, panels, and relevant tests.
- For mechanics-heavy features, search OSRS Wiki for the relevant content/mechanic pages before deciding the data model or API surface.
- For Java quality review, include thread boundaries, nullability at RuneLite API edges, render/tick hot paths, logging volume, mutable state lifetime, and tests in the inspection.
- Confirm the plugin entrypoint declared by `runelite-plugin.properties`.
- Confirm the Gradle tasks for `test`, `build`, `shadowJar`, and `run` from the actual repo.
- Check local repo instructions for required JDK. Plugin Hub expects Java 11 compatibility, but local Gradle may need a newer JDK for tool compatibility.

## Core Architecture

Typical RuneLite plugins use:

- A plugin class extending `Plugin`, annotated with `@PluginDescriptor`.
- A config interface extending `Config`.
- A `@Provides` method returning config via `ConfigManager.getConfig(...)`.
- `@Subscribe` methods in the plugin class for event handling.
- Optional overlays extending `Overlay`, `OverlayPanel`, or `WidgetItemOverlay`.
- Optional Swing side panels registered through `ClientToolbar` and `NavigationButton`.

Keep domain logic out of the plugin boundary when it grows. Prefer services/controllers/models that can be tested without starting RuneLite.

## Lifecycle

- Register overlays, toolbar buttons, subscriptions, timers, and listeners in `startUp()`.
- Unregister overlays, remove toolbar buttons, cancel scheduled work, clear collections, and shut down executors in `shutDown()`.
- Do not block startup or shutdown.
- Do not wait indefinitely for executor termination.
- Ensure UI refresh paths are called after model mutations.

## Events And State Tracking

- Prefer event subscribers over polling:
  - object/NPC spawn and despawn events for scene state
  - chat message events for chat parsing
  - game state, var, widget, item, and menu events where appropriate
- Maintain plugin-owned collections for tracked entities instead of scanning the whole scene each tick.
- Resolve game state on the client thread when touching `client`.
- Keep parsing and calculation code separate enough to unit test with synthetic events or plain strings.

## Config And Persistence

- Use `@ConfigGroup` with a specific stable group name.
- Use `@ConfigItem` keys as durable storage identifiers; changing them requires migration.
- Use `ConfigManager` for lightweight settings and hidden config values.
- Store larger plugin-owned files under `RuneLite.RUNELITE_DIR/<plugin-name>/`.
- Derive profile-specific filenames from available profile information when data must follow a RuneLite profile.
- Use structured serializers such as Gson for JSON; register adapters needed by domain types such as `Instant`.

## HTTP And JSON

- Inject `OkHttpClient` and use OkHttp for all HTTP.
- Prefer asynchronous `enqueue()` calls so network work runs off the client thread.
- Use `clientThread.invoke()` when an OkHttp callback must call RuneLite client APIs.
- Inject `Gson`; use `gson.newBuilder()` for custom adapters or options.
- Keep third-party server features opt-in and add the required IP warning to config.

## UI And Overlays

- Keep Swing panels mostly passive: render state, expose components/models, and forward user actions.
- Let controllers or plugin services own mutation and refresh orchestration.
- Use Swing table models directly for tabular state and test them without starting RuneLite.
- Run Swing mutations on the EDT when tests or code paths require it.
- Keep overlay rendering cheap. Avoid parsing, file I/O, network I/O, or whole-scene searches inside `render()`.

## API Usage

- Read [runelite-api.md](runelite-api.md) when changing game-state, event, widget, menu, item-container, coordinate, var, cache, or ID-heavy behavior.
- Prefer `net.runelite.api.gameval` constants such as `ItemID`, `ObjectID`, `NpcID`, `InventoryID`, `InterfaceID`, `VarbitID`, `VarPlayerID`, `VarClientID`, `SpriteID`, `DBTableID`, `EnumID`, `ParamID`, and `StructID` where available.
- For widgets, pass gameval component IDs directly, for example `client.getWidget(InterfaceID.DomEndLevelUi.LOOT_VALUE)`, rather than manually combining group and child IDs or using deprecated widget constants.
- Prefer `ItemContainer`, vars, cache definitions, and event payloads over parsing widget/menu text when they expose the same state.
- Use `LinkBrowser` for opening URLs.
- Use RuneLite resource loading patterns that work from packaged jars. Load plugin resources with `getClass().getResourceAsStream(...)` or equivalent classpath resource APIs; do not assume loose files next to source code.

## Dependencies And Packaging

- Do not add RuneLite client transitive dependencies directly when the client already provides them.
- Keep dependencies conservative and review-friendly.
- Keep source Java-only for Plugin Hub.
- Keep `runelite-plugin.properties` aligned with the actual plugin class.
- Do not add `META-INF/services/net.runelite.client.plugins.Plugin`.
