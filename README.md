# RuneLite Plugin Developer Skill

Codex skill and plugin package for building, reviewing, testing, and publishing Java 11 RuneLite external plugins for Old School RuneScape.

The skill focuses on RuneLite Plugin Hub constraints, Jagex third-party client guidelines, RuneLite API usage, Java quality, and practical plugin development workflow.

## Contents

- `SKILL.md` - direct skill entrypoint for local skill installation or copying.
- `agents/openai.yaml` - optional Codex UI metadata for the direct skill.
- `references/` - supporting RuneLite, OSRS, API, quality, and testing guidance loaded only when needed.
- `.codex-plugin/plugin.json` - Codex plugin manifest.
- `skills/runelite-plugin-developer/` - canonical plugin-packaged copy of the skill.

## Use as a Skill

Install or copy this repository root as a Codex skill folder. Codex discovers the skill from `SKILL.md` and can invoke it as:

```text
$runelite-plugin-developer
```

## Use as a Plugin

This repository also includes a Codex plugin manifest:

```text
.codex-plugin/plugin.json
```

The manifest points at:

```text
skills/runelite-plugin-developer/
```

Use that layout when adding this repository to a Codex plugin marketplace or when packaging it with other reusable Codex workflows.

## License

Apache-2.0
