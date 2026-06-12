# RuneLite Plugin Developer Skill

Codex plugin package for building, reviewing, testing, and publishing Java 11 RuneLite external plugins for Old School RuneScape.

The skill focuses on RuneLite Plugin Hub constraints, Jagex third-party client guidelines, RuneLite API usage, Java quality, and practical plugin development workflow.

## Repository Description

Codex skill and plugin for building, reviewing, and testing Java 11 RuneLite external plugins for OSRS with Plugin Hub and Jagex guideline checks.

## What It Helps With

- Building RuneLite external plugins for Old School RuneScape
- Reviewing plugin changes for RuneLite Plugin Hub readiness
- Choosing the right RuneLite API surface for game state, widgets, overlays, menus, and events
- Keeping Java code compatible with Java 11
- Avoiding Jagex third-party client guideline violations

## Contents

- `.codex-plugin/plugin.json` - Codex plugin manifest.
- `skills/runelite-plugin-developer/SKILL.md` - skill entrypoint.
- `skills/runelite-plugin-developer/agents/openai.yaml` - optional Codex UI metadata.
- `skills/runelite-plugin-developer/references/` - supporting RuneLite, OSRS, API, quality, and testing guidance loaded only when needed.

## Use as a Skill

Install or copy `skills/runelite-plugin-developer/` as a Codex skill folder. Codex discovers the skill from that folder's `SKILL.md` and can invoke it as:

```text
$runelite-plugin-developer
```

## Use as a Plugin

This repository includes a Codex plugin manifest:

```text
.codex-plugin/plugin.json
```

The manifest points at the `skills/` directory:

```text
skills/runelite-plugin-developer/
```

Use this layout when adding the repository to a Codex plugin marketplace or packaging it with other reusable Codex workflows.

## Suggested GitHub Topics

```text
codex
codex-skill
codex-plugin
openai-codex
agent-skills
runelite
runelite-plugin
osrs
old-school-runescape
java
java11
plugin-hub
jagex
developer-tools
ai-agents
```

## License

Apache-2.0
