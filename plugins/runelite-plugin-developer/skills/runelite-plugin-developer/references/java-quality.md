# Java Quality For RuneLite Plugins

Use this reference when writing or reviewing Java logic, concurrency, performance-sensitive paths, logging, resource handling, or tests in RuneLite external plugins.

## Sources And Adaptation

- Adapt general Java review ideas from `decebals/claude-code-java` skills such as `java-code-review`, `concurrency-review`, `performance-smell-detection`, `clean-code`, `logging-patterns`, and `test-quality`.
- Treat that material as inspiration only. RuneLite plugins have stricter local constraints: Java 11 source compatibility, RuneLite client thread rules, Swing EDT rules, overlay render hot paths, Plugin Hub policy, and no game automation.
- Ignore Java 21/25, Spring, JPA, REST service, Maven, and enterprise backend guidance unless the repository actually uses those technologies.
- Prefer the repository's existing style, test stack, and helper APIs over introducing a new review framework.

## Java 11 Compatibility

- Keep source compatible with the plugin's configured Java version. RuneLite external plugins commonly target Java 11.
- Do not introduce records, text blocks, pattern matching, sealed types, virtual threads, structured concurrency, `ScopedValue`, or APIs added after the configured target.
- Use `var` only if the project already uses it and readability remains clear. Explicit types are usually better at RuneLite API boundaries.
- Prefer simple Java collections and immutable value snapshots over framework-heavy abstractions.
- Use try-with-resources for streams and closeable resources.

## RuneLite API Boundaries

- Assume `Client`, `Widget`, `ItemContainer`, `Player`, `NPC`, `TileObject`, `WorldView`, and scene objects can be null, unloaded, hidden, stale, or no longer in the same game state.
- Read RuneLite API state at the boundary, convert it into plain domain values, then let pure code work on those values.
- On unavailable game state, prefer hiding overlays, clearing transient state, or returning an empty result over throwing.
- Clear or rebuild cached state on `GameStateChanged`, world hop, login/logout, region load, config changes, and plugin shutdown when relevant.
- Avoid storing live RuneLite objects longer than necessary. Prefer IDs, names, world points, local points, indexes, quantities, timestamps, and immutable snapshots.

## Threading And Concurrency

- Access `Client` and most RuneLite game state from the client thread. Use `ClientThread` when work must be synchronized with game state.
- Access Swing panels and Swing components on the Event Dispatch Thread. Use `SwingUtilities.invokeLater` for UI updates from callbacks.
- Run HTTP, disk, and expensive parsing work off the client thread and off the EDT.
- Do not block `startUp`, `shutDown`, event subscribers, overlay render methods, or Swing listeners with network calls, sleeps, or heavy loops.
- Avoid `Thread.sleep` in plugin logic. Model delayed behavior with events, scheduled executors, timers, or state machines where appropriate.
- Cancel callbacks, timers, subscriptions, futures, and executors in `shutDown`.
- When async work finishes, re-check plugin state, game state, config, and nullability before applying results.

## Performance Hot Paths

- Treat overlay render methods, `ClientTick`, `GameTick`, menu entry processing, widget traversal, and scene scans as hot paths.
- Keep overlay render methods focused on drawing from already-computed state.
- Cache images, fonts, colors, strokes, regex patterns, static strings, and layout measurements when they are reused.
- Avoid repeated full inventory, widget, NPC, object, or scene scans on every render frame unless the data set is small and bounded.
- Prefer straightforward loops in hot paths when they avoid avoidable allocations or improve clarity.
- Avoid logging, network calls, disk I/O, config writes, image decoding, JSON parsing, and large allocations in render and tick-heavy code.
- Optimize after understanding frequency and cost. Do not trade correctness or maintainability for speculative micro-optimizations.

## State And Collections

- Keep mutable state private to the plugin or helper that owns it.
- Prefer event-maintained maps and sets keyed by stable values such as item ID, widget ID, actor index, world point, local point, or varbit ID.
- Use immutable snapshots when passing state to overlays, panels, or tests.
- Bound caches by game state, world, region, config, or explicit size when data can grow.
- Remove stale entries when the game world changes, an actor despawns, a widget closes, or a tracked item disappears.

## Logging

- Use SLF4J parameterized logging instead of string concatenation in log calls.
- Default routine diagnostics to `debug`.
- Avoid per-frame, per-menu-entry, or per-tick `info` logs.
- Do not log secrets, tokens, local profile paths, or unnecessarily detailed player data.
- Include enough context for rare failures: IDs, widget group/child, item ID, game state, world point, config mode, or endpoint category.
- Log expected missing game state sparingly, or not at all, when it is part of normal RuneLite operation.

## Tests

- Put business logic in pure helpers that can be tested without a live RuneLite client.
- Use Mockito or local fakes for RuneLite boundaries, but prefer real simple domain objects for pure logic.
- For behavior driven by live in-game event sequences, combine small unit tests with passive capture/replay fixtures that feed the same domain services offline.
- Cover null, hidden, unloaded, empty container, missing widget, logout, region-change, and config-disabled cases.
- Test state transitions around `startUp`, `shutDown`, `GameStateChanged`, `GameTick`, `ItemContainerChanged`, and config changes when those drive behavior.
- Keep the repository's existing test stack. Do not force JUnit 5, AssertJ, or new test frameworks into a JUnit 4 plugin without a clear reason.
- Do not require live RuneLite, live OSRS login, user profile files, network access, or real Plugin Hub infrastructure for automated tests.

## Review Checklist

- Does the code use the RuneLite API on the right thread and handle unavailable game state?
- Does the implementation preserve Java 11 compatibility?
- Are render and tick handlers free of blocking work and obvious allocation churn?
- Is mutable state cleared on logout, world hop, region load, config change, or shutdown where relevant?
- Are logs useful, quiet by default, and free of sensitive data?
- Are tests focused on the plugin's real behavior rather than only constructor or getter coverage?
