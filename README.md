# pnpm-upgrade

A [Claude Code](https://claude.com/claude-code) skill for upgrading pnpm versions (v9 to v10+). Includes breaking changes documentation, recommended settings, and guidance for investigating build scripts.

## Installation

### Option 1: As a Plugin

Clone this repo and run Claude Code with the plugin directory:

```bash
git clone https://github.com/dheid16/pnpm-upgrade-plugin.git
claude --plugin-dir ./pnpm-upgrade-plugin
```

### Option 2: Copy as a Personal Skill

Create the skill directory and file:

```bash
mkdir -p ~/.claude/skills/pnpm-upgrade
```

Then copy the content from `/skills/pnpm-upgrade/SKILL.md` into your own local directory (`~/.claude/skills/pnpm-upgrade/SKILL.md`):

## Usage

Once installed, the skill activates automatically when you ask Claude Code about:

- Upgrading pnpm versions
- pnpm breaking changes
- Configuring pnpm after a major upgrade
- Investigating blocked build scripts
