# RuneLite API Usage

Use this reference before implementing or reviewing plugin behavior that reads game state, reacts to game/client events, renders overlays, inspects widgets, handles menu entries, reads item containers, checks vars, works with coordinates, or needs IDs/constants.

If the task depends on the broader OSRS/RuneLite/user boundary or OSRS mechanics, read [runelite-concepts.md](runelite-concepts.md) first and search OSRS Wiki for game-context pages.

Sources:

- RuneLite API Javadocs: https://static.runelite.net/runelite-api/apidocs/index.html
- RuneLite API package summary: https://static.runelite.net/runelite-api/apidocs/net/runelite/api/package-summary.html
- RuneLite events package: https://static.runelite.net/runelite-api/apidocs/net/runelite/api/events/package-summary.html
- RuneLite widgets package: https://static.runelite.net/runelite-api/apidocs/net/runelite/api/widgets/package-summary.html
- RuneLite Developer Guide: https://github.com/runelite/runelite/wiki/Developer-Guide

## Lookup Workflow

1. Identify the gameplay surface: actor, item container, widget, menu, scene tile/object, coordinate, var, chat, inventory/equipment, cache definition, price/sprite, or external service.
2. Inspect the target repo's resolved RuneLite version before relying on memory:
   - `./gradlew -q dependencies --configuration compileClasspath`
   - Search the resolved artifacts when Javadocs are incomplete: `jar tf ~/.gradle/caches/modules-2/files-2.1/net.runelite/runelite-api/<version>/**/runelite-api-<version>.jar`
   - Prefer source jars for exact method signatures and examples when present under Gradle cache.
3. Search OSRS Wiki for game-mechanics context when the API type alone does not explain player-facing behavior.
4. Search the official Javadocs or local source for the exact type before adding custom parsing or magic numbers.
5. Prefer event-driven state capture, then read current state through `Client` only when the event does not carry enough data.
6. Keep all `Client` access on the client thread. Use `ClientThread.invoke()` when crossing from OkHttp callbacks, Swing listeners, executor work, or tests.
7. Check the deprecated list and replacement notes. Avoid deprecated widget/ID/var helpers such as `WidgetInfo`, `ComponentID`, root `InventoryID`, `Varbits`, `VarPlayer`, `VarClientInt`, and `VarClientStr`; prefer `net.runelite.api.gameval` and newer APIs when they exist in the resolved dependency.

## API Map

- Core game state: `Client`, `GameState`, `WorldView`, `Scene`, `Tile`, `Player`, `NPC`, `Actor`, `TileObject`, `GameObject`, `GroundObject`, `WallObject`, `DecorativeObject`, `Projectile`, `GraphicsObject`.
- Events: `GameStateChanged`, `GameTick`, `ClientTick`, `PostClientTick`, spawn/despawn events for objects/NPCs/players/items, `ItemContainerChanged`, `StatChanged`, `VarbitChanged`, `ChatMessage`, `WidgetLoaded`, `WidgetClosed`, `MenuOpened`, `MenuEntryAdded`, `PostMenuSort`, `MenuOptionClicked`.
- Widgets and UI: `Widget`, `WidgetItem`, `WidgetItemOverlay`, `WidgetLoaded`, `WidgetClosed`, `client.getWidget(...)`, widget bounds/canvas bounds, dynamic/static/nested children, item ID/quantity, text, hidden state.
- Menus: `Menu`, `MenuEntry`, `MenuAction`, `MenuOpened`, `MenuEntryAdded`, `PostMenuSort`, `MenuOptionClicked`. Read menu data for overlays and context; avoid adding entries that send server actions unless Plugin Hub rules clearly permit it.
- Items and containers: `Item`, `ItemContainer`, `EquipmentInventorySlot`, `ItemComposition`, `ItemContainerChanged`, `client.getItemContainer(...)`, `ItemManager` for prices/images/canonicalization.
- IDs and constants: prefer `net.runelite.api.gameval.ItemID`, `NpcID`, `ObjectID`, `InventoryID`, `InterfaceID`, `VarbitID`, `VarPlayerID`, `VarClientID`, `SpriteID`, `DBTableID`, `EnumID`, `ParamID`, `StructID`, `AnimationID`, and `ScriptID` over handwritten integers.
- Coordinates and geometry: `WorldPoint`, `WorldArea`, `LocalPoint`, `Angle`, `Direction`, `Perspective`, `Point`, `Polygon`, `Shape`, `WorldView`. Convert explicitly between world, local, scene, minimap, and canvas coordinates.
- Vars and client state: use `VarbitChanged` plus `client.getVarbitValue(...)`, `client.getVarpValue(...)`, and `gameval` var constants. Treat root var constant classes as legacy when replacements are available.
- Cache definitions: `client.getItemDefinition`, `getNpcDefinition`, `getObjectDefinition`, `getEnum`, `getStructComposition`, `getDBTableRows`, `getDBTableField`, and post-composition events for cache-driven metadata.
- Chat and messages: `ChatMessage`, `ChatMessageType`, `MessageNode`, `client.addChatMessage(...)`. Do not programmatically inject user chat input or alter outgoing chat.
- Client services outside `net.runelite.api`: use injected `ItemManager`, `SpriteManager`, `NPCManager`, `WorldService`, `ClientThread`, `OverlayManager`, `ClientToolbar`, `ConfigManager`, `KeyManager`, `ChatMessageManager`, `Notifier`, `OkHttpClient`, and `Gson` when those match the task and are allowed by Plugin Hub rules.

## Selection Heuristics

- If the feature observes something changing in game, look for an event first. Examples: `ItemContainerChanged` for inventories, `NpcSpawned`/`NpcDespawned` for NPC tracking, `GameObjectSpawned`/`GameObjectDespawned` for scene objects, `VarbitChanged` for state toggles, `ChatMessage` for chat-derived state.
- If the feature draws on an existing item slot, use `WidgetItemOverlay` and `WidgetItem` bounds. If it draws world-space markers, use an `Overlay` with `Perspective` helpers and cached points. If it draws panel text, use `OverlayPanel`.
- If the feature needs a widget, use `gameval.InterfaceID` nested component constants when available. Use `client.getWidget(InterfaceID.SomeInterface.SOME_COMPONENT)` rather than combining group and child IDs by hand.
- If the feature needs an item, object, NPC, sprite, var, script, enum, struct, or DB table ID, search `net.runelite.api.gameval` first. Avoid maintaining parallel constant lists unless the API has no constant.
- If the feature needs prices, icons, noted/unnoted mappings, variations, or item stats, look for `ItemManager` or client game services before hitting external APIs or parsing wiki data.
- If a visible widget mirrors an item container, prefer the container for item IDs/quantities and use the widget only for layout, selection, hover, and bounds.
- If a feature needs account/profile persistence, use `ConfigManager` for lightweight config and `RuneLite.RUNELITE_DIR` for plugin files; do not infer identity from player names when account/profile APIs are available.

## Anti-Patterns

- Do not scan all tiles, NPCs, players, widgets, or menu entries every tick if spawn/despawn, widget, menu, or var events can maintain state.
- Do not parse colored widget/menu text when `MenuEntry`, `Widget`, `ItemContainer`, `Varbit`, or cache definitions expose the underlying value.
- Do not use reflection to reach client internals. If the public API does not expose a value, treat that as a design constraint.
- Do not depend on deprecated ID classes or `WidgetInfo` when `gameval` constants are available in the resolved dependency.
- Do not call `Client` from arbitrary background threads.
- Do not perform network, file I/O, expensive image generation, cache searches, or whole-scene scans inside overlay `render()`.

## Practical Source Searches

- Find current API classes in the resolved jar: `jar tf <runelite-api-jar> | rg '^net/runelite/api/(events|widgets|coords|gameval)/'`
- Inspect a class signature: `javap -classpath <runelite-api-jar> net.runelite.api.Client`
- Search local source jars for examples: `jar tf <client-sources-jar> | rg 'net/runelite/client/plugins/.+Plugin.java'`
- Search the target plugin for avoidable raw IDs: `rg '\\b(ItemID|NpcID|ObjectID|InventoryID|InterfaceID|VarbitID|VarPlayerID|VarClientID|WidgetInfo|ComponentID)\\b|\\b\\d{3,}\\b' src/main/java`
