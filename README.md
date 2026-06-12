# RuneLite Plugin Developer Marketplace

Codex plugin marketplace for building, reviewing, testing, and publishing Java 11 RuneLite external plugins for Old School RuneScape.

The skill focuses on RuneLite Plugin Hub constraints, Jagex third-party client guidelines, RuneLite API usage, Java quality, and practical plugin development workflow.

## Repository Description

Codex plugin marketplace for a RuneLite external plugin development skill focused on OSRS, Java 11, Plugin Hub compliance, and Jagex guideline checks.

## Suggested Repository Name

`runelite-plugin-developer-marketplace`

The current repository can still be used as-is. If you rename it, update the install command and plugin metadata URLs to the new repository name.

## What It Helps With

- Building RuneLite external plugins for Old School RuneScape
- Reviewing plugin changes for RuneLite Plugin Hub readiness
- Choosing the right RuneLite API surface for game state, widgets, overlays, menus, and events
- Keeping Java code compatible with Java 11
- Avoiding Jagex third-party client guideline violations

## Contents

- `.agents/plugins/marketplace.json` - Codex marketplace catalog.
- `plugins/runelite-plugin-developer/.codex-plugin/plugin.json` - Codex plugin manifest.
- `plugins/runelite-plugin-developer/skills/runelite-plugin-developer/SKILL.md` - skill entrypoint.
- `plugins/runelite-plugin-developer/skills/runelite-plugin-developer/agents/openai.yaml` - optional Codex UI metadata.
- `plugins/runelite-plugin-developer/skills/runelite-plugin-developer/references/` - supporting RuneLite, OSRS, API, quality, and testing guidance loaded only when needed.

## Use as a Skill

Install or copy `plugins/runelite-plugin-developer/skills/runelite-plugin-developer/` as a Codex skill folder. Codex discovers the skill from that folder's `SKILL.md` and can invoke it as:

```text
$runelite-plugin-developer
```

## Add as a Codex Marketplace

Add this repository as a Codex plugin marketplace:

```bash
codex plugin marketplace add pk-enjoyer/runelite-plugin-developer-skill --ref main
```

Then open the Codex plugin directory, select `pk-enjoyer Codex Plugins`, and install `RuneLite Plugin Developer`.

## Plugin Layout

The marketplace entry points at:

```text
plugins/runelite-plugin-developer/
```

That plugin manifest points at its bundled `skills/` directory.

## License

Apache-2.0
