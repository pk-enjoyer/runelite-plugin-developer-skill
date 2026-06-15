# Plugin Hub Research

Use RuneLite core and Plugin Hub research to avoid duplicating existing plugin functionality, learn current implementation patterns, and find realistic examples for RuneLite APIs. Treat existing plugins as examples, not authority: Plugin Hub rules, Jagex guidelines, and RuneLite API docs still win.

## When To Search

- Before creating a new external plugin or adding a broad feature.
- When a user asks whether a plugin already exists, whether a feature is acceptable for Plugin Hub, or how other plugins implement similar behavior.
- When using unfamiliar APIs for widgets, overlays, menus, item containers, varbits, world/scene data, HTTP, persistence, or config.
- When reviewing a change that may duplicate a Plugin Hub plugin or copy a risky pattern from older code.

## Search Strategy

1. Generate search terms from the player-facing feature, OSRS terms, RuneLite API names, events, widgets, item/object/NPC names, and likely plugin names.
2. Search `runelite/runelite` core plugins first for broad/common features. Built-in plugins are not listed in `runelite/plugin-hub`, but they are often the best prior art and the current API example.
3. Search `runelite/plugin-hub` manifests next. Manifests identify accepted external Plugin Hub plugins and usually point to source repositories through `repository=`.
4. Inspect candidate plugin manifests for `displayName`, `description`, `tags`, `plugins`, `repository`, `commit`, and `build`.
5. Follow the source repository and search Java files for `@PluginDescriptor`, relevant event subscribers, injected services, config keys, overlays, panels, and model classes.
6. Compare behavior before using code as an example. Prefer current RuneLite core code for shared services and API patterns; prefer recent, simple, narrowly scoped Plugin Hub plugins for external-plugin structure. Watch for stale APIs, magic IDs, broad widget edits, high-frequency scanning, reflection, fresh `Gson` creation, or patterns that current Plugin Hub tooling may reject.
7. Summarize relevant prior art in the final answer or implementation notes: similar built-in and Plugin Hub plugins found, whether the request appears duplicative, which examples informed the approach, and why the final implementation differs.

## Practical Commands

Use GitHub search when available:

```bash
gh search code '"@PluginDescriptor" "Highlight NPCs" repo:runelite/runelite'
gh search code '"NpcOverlayService" "registerHighlighter" repo:runelite/runelite'
gh search code '"displayName" "loot" repo:runelite/plugin-hub'
gh search code '"repository=" "clue" repo:runelite/plugin-hub'
gh search code '"@PluginDescriptor" "ItemContainerChanged" language:Java'
```

For local/offline inspection, clone or update RuneLite and Plugin Hub outside the target plugin repository, then search core plugins and manifests:

```bash
git clone --depth 1 https://github.com/runelite/runelite /tmp/runelite
git clone --depth 1 https://github.com/runelite/plugin-hub /tmp/runelite-plugin-hub
rg -n "@PluginDescriptor|NpcOverlayService|NpcSpawned|NpcDespawned" /tmp/runelite/runelite-client/src/main/java/net/runelite/client/plugins
rg -i "loot|chest|wilderness" /tmp/runelite-plugin-hub/plugins
rg -n "^repository=|^displayName=|^description=|^tags=|^plugins=" /tmp/runelite-plugin-hub/plugins/<candidate>
```

If a source repository is found, clone it under `/tmp` or inspect it through GitHub search. Search for both domain terms and API names:

```bash
rg -n "@PluginDescriptor|Subscribe|ItemContainerChanged|Widget|Overlay|Config" /tmp/<candidate-plugin>
rg -n "ClientThread|OverlayManager|ConfigManager|OkHttpClient|Gson" /tmp/<candidate-plugin>
```

## Interpreting Results

- RuneLite core coverage usually means a generic external plugin would be duplicative; look for a narrower user-facing distinction before implementing.
- Plugin Hub presence means the plugin has been accepted at least at the pinned manifest commit; it does not prove the pattern is still preferred.
- The Plugin Hub repository does not provide a reliable popularity ranking. Use repository stars, recent commits, issue activity, and community visibility only as rough signals when choosing examples.
- If a candidate plugin overlaps heavily with the user's idea, recommend differentiating the feature, contributing upstream, or explaining the user-facing distinction before implementing.
- Avoid copying code wholesale. Use examples to identify API surfaces, lifecycle placement, and edge cases, then write code that fits the target repository.
