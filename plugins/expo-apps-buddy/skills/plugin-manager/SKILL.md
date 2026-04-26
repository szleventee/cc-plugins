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

The plugin source is a real **git working copy** of `szleventee/cc-plugins` that lives on the user's Mac at:

```
~/.claude/plugins/marketplaces/cc-plugins/
├── .claude-plugin/marketplace.json          ← marketplace index
└── plugins/expo-apps-buddy/
    ├── .claude-plugin/plugin.json           ← plugin manifest (version)
    ├── README.md
    └── skills/
        ├── beginner-guide/SKILL.md
        ├── custom-ngrok-tunnel/SKILL.md
        ├── expo-setup/SKILL.md
        └── plugin-manager/SKILL.md          ← this file
```

**GitHub repo:** `https://github.com/szleventee/cc-plugins`

This means **anywhere the plugin is installed**, you can:
1. Edit the SKILL.md files directly
2. `git commit` and `git push` them to GitHub
3. Reload the plugin

…and the changes will land on every Mac where the plugin is installed (after each one runs the reload commands). **You don't need to be on a "special" machine or in a "special" project to fix the plugin** — fix it from wherever you noticed the issue.

### Verify the source is there before editing

```bash
ls ~/.claude/plugins/marketplaces/cc-plugins/plugins/expo-apps-buddy/skills/
# Should list: beginner-guide  custom-ngrok-tunnel  expo-setup  plugin-manager

cd ~/.claude/plugins/marketplaces/cc-plugins && git remote -v
# Should show: origin  https://github.com/szleventee/cc-plugins.git (fetch/push)
```

If `git remote -v` shows nothing or the directory is missing, the plugin was probably installed from a different marketplace — stop and ask the user.

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

**IMPORTANT: Always run the marketplace update FIRST, then the plugin update.** The marketplace update re-fetches the index from GitHub so Claude Code knows a new version exists. Without it, `plugin update` will say "already at latest version" even though a newer one was pushed.

### From the terminal (CLI):

```bash
# Step 1: Re-fetch the marketplace index from GitHub (MUST be first)
claude plugin marketplace update cc-plugins

# Step 2: Update the plugin to the new version
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

## When to fix the plugin vs. the current project

While working on any user's app, you'll often notice friction or missing behavior. Decide where the fix belongs:

**Fix in the plugin** (here, in `~/.claude/plugins/marketplaces/cc-plugins/`) when the issue is:
- A missing default in `expo-setup` (e.g. a package every project should have)
- A behavior pattern that should apply to every beginner session (e.g. how Claude restarts Metro)
- A common pitfall mentioned in `beginner-guide` known-issues
- Anything that, if left only in the project's `CLAUDE.md`, would have to be re-discovered in the next project

**Fix in the project** (e.g. `<project>/CLAUDE.md`) when the issue is:
- Specific to this app's domain (custom ngrok subdomain, particular meme list, this project's API key)
- A workaround that's still being validated and isn't yet generalizable
- Something the user explicitly says is "just for this project"

### The conversation flow

When you spot something that smells plugin-shaped:

1. Make the fix work in the current project first (so the user is unblocked)
2. Once the user confirms it works on their phone/web, propose: *"This feels like a plugin-level fix — want me to bake it into expo-apps-buddy v1.x.y so every future project gets it for free?"*
3. If yes: edit the SKILL.md files, bump version, commit, ask the user to push (or push if you have auth), run reload commands
4. After the plugin gains the behavior, **strip the duplicated content from the project's CLAUDE.md** — it's now inherited.

## CLAUDE.md philosophy (project-level)

Once a behavior moves into the plugin, the project's `CLAUDE.md` should NOT repeat it. Project-level `CLAUDE.md` files should stay lean and contain only:

- 🎯 What the app is and what it does
- 📍 Where the project currently is (current state, what's working, what's WIP)
- 📁 Project-specific file map (only the files that matter for this app)
- 🔑 Project-specific environment (custom ngrok domain, API keys location, etc.)
- 🐛 Project-specific known bugs and their fixes
- 🎬 Project-specific user flows or behavior

The project's `CLAUDE.md` should NOT contain:
- ❌ How auto-save / git commits work (plugin handles this)
- ❌ Beginner-friendly translation tables (plugin handles this)
- ❌ Generic "Claude as dev server caretaker" instructions (plugin handles this)
- ❌ Generic Metro/ngrok start/restart commands (plugin handles this)
- ❌ How to bump versions / publish the plugin (this skill handles that)

Keeping project `CLAUDE.md` lean is also a good way to **measure how well the plugin works**: the more empty / minimal a project's `CLAUDE.md` is, the more the plugin is doing its job.
