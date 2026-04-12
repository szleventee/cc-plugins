---
name: plugin-manager
description: Manage the expo-apps-buddy plugin — reload after updates, bump versions, publish changes. Use this skill when the user asks to update, reload, or publish the plugin, or when you've made changes to any skill and need to bump the version.
argument-hint: "[reload | bump | publish]"
allowed-tools:
  - Bash
  - Read
  - Edit
  - Glob
  - Grep
---

# Plugin Manager

Manages the expo-apps-buddy Claude Code plugin — versioning, publishing, and reloading.

## Plugin File Locations

- **Marketplace index:** `.claude-plugin/marketplace.json`
- **Plugin manifest:** `plugins/expo-apps-buddy/.claude-plugin/plugin.json`
- **Skills:** `plugins/expo-apps-buddy/skills/`
- **GitHub repo:** `szleventee/cc-plugins`

## Version Bumping

**Every time you change any skill, config, or behavior in this plugin, you MUST bump the version.** This is how users receive updates in the Claude Code desktop app — if you don't bump, they won't see the change.

Bump the `version` field in BOTH files:
1. `plugins/expo-apps-buddy/.claude-plugin/plugin.json`
2. `.claude-plugin/marketplace.json` (the `expo-apps-buddy` entry)

### Semver Rules

- **Patch** (1.2.1 → 1.2.2): Bug fixes, typo corrections, minor wording changes
- **Minor** (1.2.0 → 1.3.0): New features, new skills, behavior changes, new capabilities
- **Major** (1.0.0 → 2.0.0): Breaking changes (rarely needed)

### How to Bump

1. Read both files to check the current version
2. Decide patch or minor based on the change
3. Update the `version` field in both files to the same new version
4. Commit and push

## Publishing Changes

After making changes and bumping the version:

```bash
# Stage, commit, and push
git add -A
git commit -m "<description of changes>"
git push origin main
```

Then tell the user to run the reload commands (see below), or run them yourself if you're in the same session.

## Reloading the Plugin

After pushing updates to GitHub, the user (or you) need to tell Claude Code to fetch the new version.

### From the terminal (CLI):

```bash
claude plugin marketplace update cc-plugins
claude plugin update expo-apps-buddy@cc-plugins
```

### From inside a Claude Code session (slash commands):

```
/plugin marketplace update cc-plugins
/plugin update expo-apps-buddy@cc-plugins
/reload-plugins
```

### Nuclear option (if update doesn't work):

```bash
claude plugin uninstall expo-apps-buddy@cc-plugins
claude plugin marketplace update cc-plugins
claude plugin install expo-apps-buddy@cc-plugins
```

## Common Tasks

### "Reload the plugin" / "Update the plugin"

1. Make sure all changes are committed and pushed to GitHub
2. Run the reload commands above

### "I changed a skill, now what?"

1. Bump the version (see Version Bumping above)
2. Commit and push
3. Run the reload commands

### "Add a new skill"

1. Create `plugins/expo-apps-buddy/skills/<skill-name>/SKILL.md` with frontmatter
2. Bump the minor version
3. Commit and push
4. Run the reload commands
