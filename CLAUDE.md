# cc-plugins

Claude Code plugin marketplace by szleventee.

## Version Bumping

When making ANY change to plugin skills, config, or behavior, always bump the version in BOTH files:
- `.claude-plugin/marketplace.json` (the plugin entry's `version` field)
- `plugins/<plugin-name>/.claude-plugin/plugin.json` (`version` field)

Use semver: patch (1.1.1) for bug fixes, minor (1.2.0) for new features/behavior changes. This is how users receive updates in the Claude Code desktop app — if you don't bump, they won't see the update.

## Structure

```
.claude-plugin/marketplace.json    — marketplace index (lists all plugins)
plugins/
  expo-apps-buddy/
    .claude-plugin/plugin.json     — plugin manifest
    skills/                        — skill definitions (SKILL.md per skill)
```

## Adding a New Plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json` with at least `name` and `version`
2. Add a `skills/` directory with at least one `<skill-name>/SKILL.md`
3. Add an entry to `.claude-plugin/marketplace.json` under `plugins` with `name`, `source`, and `version`
