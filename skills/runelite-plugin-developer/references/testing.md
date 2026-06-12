# RuneLite Plugin Testing

Use this reference when adding behavior, reviewing risk, changing Gradle/test setup, or finishing a plugin task.
Read [java-quality.md](java-quality.md) alongside this file when test quality or Java logic review is part of the task.

Sources:

- RuneLite Developer Guide: https://github.com/runelite/runelite/wiki/Developer-Guide
- RuneLite Plugin Hub README: https://github.com/runelite/plugin-hub
- RuneLite example-plugin AGENTS.md: https://github.com/runelite/example-plugin/blob/master/AGENTS.md
- RuneLite Rejected or Rolled Back Features: https://github.com/runelite/runelite/wiki/Rejected-or-Rolled-Back-Features
- Jagex Third-Party Client Guidelines: https://secure.runescape.com/m=news/third-party-client-guidelines?oldschool=1
- RuneLite wiki "Using Jagex Accounts": https://github.com/runelite/runelite/wiki/Using-Jagex-Accounts

## Automated Tests

- Use `./gradlew test` as the default automated verification command unless the repo documents a different command.
- Use `./gradlew build` when compile/package integration matters.
- Use the repo's documented JDK command if Java tooling compatibility requires it.
- Keep tests Java 11 compatible.
- Follow the repo's current test framework. External plugin templates commonly use JUnit 4, Mockito, `net.runelite:client`, and `net.runelite:jshell`.

## Test Boundaries

- Prefer domain unit tests for parsers, calculators, session models, formatters, serializers, and settlement logic.
- Use boundary unit tests for plugin event handlers with mocked RuneLite services and synthetic events.
- Use view/model tests for Swing table models and formatting without launching RuneLite.
- Mock `Client`, config interfaces, `ClientToolbar`, `OverlayManager`, panels, and other RuneLite boundaries.
- Prefer real domain objects over mocks for business state.
- Stub config values explicitly; Mockito may not call interface default methods unless configured.

## Capture And Replay Fixtures

Use capture/replay testing when plugin behavior depends on real in-game event sequences but automating gameplay would be disallowed, flaky, or too expensive.

The safe shape is:

```text
manual gameplay in a dev client
  -> passive RuneLite event listener
  -> narrow DTO / domain event fixture
  -> offline unit test replay
  -> assertions on plugin state, calculations, panel models, or overlay models
```

This complements unit testing; it does not replace it. Keep pure unit tests for calculators, parsers, formatters, and state transitions. Add replay fixtures for realistic ordering, missing data, tick timing, and multi-event interactions that are hard to invent accurately by hand.

### Capture Boundaries

- Capture only from passive RuneLite event subscribers during manual user play.
- Do not inject input, click, type, select menu entries, log in automatically, pathfind, fight, skill, or otherwise create game actions for the player.
- Do not run automated tests against a live RuneLite client, live OSRS login, or the user's live profile.
- Prefer keeping capture tools in test/dev helper code. If capture code must live temporarily in the plugin, gate it behind an explicit developer-only option, disabled by default, and remove it before Plugin Hub submission unless it is a real reviewed feature.
- Store raw captures under a plugin-owned development path, then copy reviewed fixtures into `src/test/resources`.

### What To Capture

Capture the smallest stable shape needed by the plugin logic:

- event type and tick index
- game state transitions
- item container ID, item IDs, and quantities
- widget group/child/component IDs and only the text or properties actually used
- varbit/varplayer/varclient IDs and values
- NPC, object, item, projectile, animation, and graphic IDs when needed
- world point or local point only when location is part of the behavior
- chat message type and sanitized text only for plugins whose behavior depends on chat
- config values relevant to the scenario

Avoid capturing account names, display names, private messages, friends/clan/ignore data, auth tokens, local profile paths, worlds, or other player-identifying data unless the feature genuinely requires it and the fixture is anonymized.

### Fixture Design

- Use JSON fixtures or another simple structured format that can be read from `src/test/resources`.
- Version the fixture schema with a small integer field so tests can reject stale captures clearly.
- Prefer domain event names over RuneLite class names when possible, for example `ITEM_CONTAINER_CHANGED`, `GAME_TICK`, `VAR_CHANGED`, or `CHAT_MESSAGE`.
- Keep fixtures short and scenario-focused. Split long captures into named cases such as `loot-key-opened.json`, `loot-key-empty.json`, or `logout-clears-state.json`.
- Normalize noisy values before committing fixtures: timestamps, random IDs, player names, world numbers, and unrelated widget text.

Example fixture shape:

```json
{
  "schema": 1,
  "scenario": "inventory value updates after loot key opens",
  "events": [
    { "type": "GAME_STATE", "tick": 0, "state": "LOGGED_IN" },
    {
      "type": "ITEM_CONTAINER_CHANGED",
      "tick": 1,
      "containerId": 93,
      "items": [
        { "id": 995, "quantity": 12000 },
        { "id": 561, "quantity": 44 }
      ]
    },
    { "type": "GAME_TICK", "tick": 2 }
  ]
}
```

### Replay Tests

- Replay fixtures through the same pure service or state machine used by the plugin boundary.
- Keep RuneLite event classes at the edge. Map RuneLite events to domain DTOs in production, and map fixture DTOs to the same domain DTOs in tests.
- Assert behavior after meaningful checkpoints, not every event, unless the event-by-event state is the contract.
- Test lifecycle edges with replay data: logout clears state, region changes remove stale entities, missing widgets hide overlays, config-disabled paths ignore events, and duplicate events do not double-count.
- Treat replay tests as regression tests for observed game behavior. Still use manual RuneLite smoke testing for rendering, client wiring, and real in-game feel.

## Avoid In Automated Tests

- Do not start RuneLite from unit tests.
- Do not log into OSRS or automate gameplay.
- Do not hit live network services.
- Do not touch the user's live RuneLite profile directory.
- Do not depend on clipboard, dialogs, screen capture, focus state, or live UI state.
- Avoid `JOptionPane` in test paths. If needed, move the decision into domain/controller code and keep dialog calls as thin UI shells.
- Avoid tests based on wall-clock timestamps, random IDs, map iteration order, or Swing selection side effects unless that ordering or behavior is explicitly part of the contract.

## Swing Tests

- Instantiate Swing table models directly where possible.
- Assert `getValueAt`, editability, mutation behavior, and formatting.
- If Swing UI mutation must be tested, run it on the EDT with `SwingUtilities.invokeAndWait`.
- Keep Swing tests headless-friendly.

## Manual RuneLite Smoke Testing

Manual validation is required for gameplay-facing behavior. Automated checks and a clean JVM start do not prove the plugin works in game.

When finishing:

- Offer to run `./gradlew run` from the plugin root to launch the development client.
- Instruct the user to follow RuneLite's "Using Jagex Accounts" instructions: https://github.com/runelite/runelite/wiki/Using-Jagex-Accounts
- Give a concrete checklist:
  - golden path for the changed behavior
  - edge cases and failure paths
  - config toggles or persistence behavior
  - startup/shutdown cleanup where relevant
- Wait for the user to confirm in-game behavior before considering gameplay-facing work fully validated.
