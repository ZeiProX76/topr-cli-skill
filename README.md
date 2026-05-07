# topr CLI skill

Public mirror of the [topr CLI](https://www.npmjs.com/package/@speedy_devv/topr) AI skill for Claude Code, Codex, and other agents.

## Install

The recommended way is via the CLI itself:

```bash
npm install -g @speedy_devv/topr
topr skill install
```

That fetches `SKILL.md` from this repo and writes it to `~/.claude/skills/topr/SKILL.md`.

### Manual install

```bash
mkdir -p ~/.claude/skills/topr
curl -fsSL https://raw.githubusercontent.com/ZeiProX76/topr-cli-skill/main/SKILL.md \
  -o ~/.claude/skills/topr/SKILL.md
```

## Why a separate repo?

The main Topr codebase is private. This repo exists solely to host the skill file at a stable, public URL so the CLI can fetch it without any auth.
