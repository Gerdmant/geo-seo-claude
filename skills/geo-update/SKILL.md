---
name: geo-update
description: Update this GEO-SEO installation. This fork is installed as a Claude Code plugin, not by the upstream install.sh script, so updating means syncing the fork with upstream and refreshing the plugin. Use when user says "update geo", "geo-update", "aggiorna geo", or asks how to get the latest GEO-SEO changes.
allowed-tools:
  - Bash
  - Read
---

# Updating GEO-SEO

This repository is a fork of `zubair-trabzada/geo-seo-claude`, repackaged as a
Claude Code plugin. Upstream ships only `install.sh`, which copies skills into
`~/.claude/skills/` — that path is **not** used here. Do not run it.

## How this installation works

| Piece | Location |
|---|---|
| Plugin source | fork `Gerdmant/geo-seo-claude` |
| Installed copy | managed by Claude Code, path is not stable across restarts |
| Python for the scripts | `~/.claude/venvs/geo-seo-claude` (requests, beautifulsoup4, lxml, rich, flask) |

## Update procedure

Pull upstream changes into the fork, then refresh the plugin:

```bash
gh repo sync Gerdmant/geo-seo-claude
```

Then in Claude Code run `/plugin update`, or reinstall the plugin from the
marketplace entry if the update command reports nothing.

## After syncing — check these

Upstream does not know about the plugin packaging, so a sync can reintroduce
assumptions that do not hold here. Verify:

1. `.claude-plugin/marketplace.json` and `.claude-plugin/plugin.json` still exist.
2. The main skill is at `skills/geo/SKILL.md`, not `geo/SKILL.md` — upstream keeps
   it outside `skills/` where the plugin loader cannot see it.
3. No file references `~/.claude/skills/geo/` — those are installer paths and must
   read `${CLAUDE_PLUGIN_ROOT}/` instead.
4. Script invocations point at the venv interpreter, not a bare `python3`. The
   system `python3` on this machine is 3.9.6 and has no `requests` or `bs4`:

```bash
grep -rn "python3 \|~/.claude/skills/geo" skills agents
```

Empty output means the packaging survived the sync. Anything printed is a
regression to fix before using the skills.

## Rebuilding the venv

Only needed if the interpreter is missing or dependencies changed:

```bash
uv venv --python /opt/homebrew/bin/python3.13 ~/.claude/venvs/geo-seo-claude
uv pip install --python ~/.claude/venvs/geo-seo-claude/bin/python requests beautifulsoup4 lxml rich flask
```

## What upstream's installer would have done

For reference only — do not run it. It clones into `~/.claude/skills/geo`, builds
a venv there, rewrites script shebangs and patches `.md` files to point at that
venv. All of that is replaced by the plugin packaging described above.
