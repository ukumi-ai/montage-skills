# Contributing to montage-skills

This repo is three things at once, from one set of files:

- a **skills collection** (`npx skills add ukumi-ai/montage-skills`)
- a **Claude Code plugin + marketplace**
- a **ChatGPT / Codex plugin + marketplace**

The repository root *is* the plugin root. There is no `plugins/` subdirectory —
both hosts point at the root, so `skills/` stays where skills.sh expects it.

## Layout

```
.claude-plugin/
  plugin.json         Claude Code manifest
  marketplace.json    Claude Code marketplace (also read by Codex as a legacy source)
.codex-plugin/
  plugin.json         ChatGPT / Codex manifest — adds the `interface` block
.agents/plugins/
  marketplace.json    Codex-native marketplace
.mcp.json             Montage MCP server (remote, OAuth) — shared by both hosts
.app.json             ChatGPT registered-connection mapping (see "ChatGPT wiring")
skills/<name>/SKILL.md
agents/<name>.md
```

Only manifests live in `.claude-plugin/` and `.codex-plugin/`. Everything else —
`skills/`, `agents/`, `hooks/`, `assets/`, `.mcp.json` — stays at the root. Both
hosts enforce this.

## Add a skill

Create `skills/<name>/SKILL.md` with `name` and `description` frontmatter:

```markdown
---
name: my-skill
description: Use when the user wants X. Be specific — this text is the only thing the host sees when deciding whether to load the skill.
---

# What this does

Steps here.
```

That is the entire process. Both manifests point at `./skills/`, so a new
directory is picked up by both hosts with no manifest edit. Supporting files
(`reference.md`, `scripts/`) can sit alongside `SKILL.md`.

Write the `description` for a host that has not read the body. "Use when the
user wants to ingest, upload, or import a video" beats "Video ingestion skill" —
the first tells the model when to reach for it.

## Add an agent

Create `agents/<name>.md`. Frontmatter needs `name` and `description`; Claude
Code requires it, and Codex tolerates it, so always include it.

```markdown
---
name: my-agent
description: One line on what this agent is for and when to dispatch it.
---

You are ...
```

`agents/clip-scout.md` is a working example — a read-only scout that proposes
clip segments without creating them.

Claude Code also accepts `model`, `effort`, `maxTurns`, `tools`,
`disallowedTools`, `skills`, `memory`, `background`, and `isolation` (only
`"worktree"`). For security reasons `hooks`, `mcpServers`, and `permissionMode`
are rejected in plugin-shipped agents.

**If you restrict `tools`, MCP tools need their scoped plugin names.** Inside
this plugin the Montage tools are:

```
mcp__plugin_montage_montage__<tool>
```

That is `mcp__plugin_<plugin-name>_<server-name>__<tool>`, and both names here
happen to be `montage`. So `get_transcript` is
`mcp__plugin_montage_montage__get_transcript`. A `tools` list written against
the bare tool name silently matches nothing — the agent starts with no Montage
access and the failure looks like the server being down. When in doubt, omit
`tools` and state the constraint in the prompt instead.

## The MCP server and auth

`.mcp.json` points at `https://mcp.montage.app/mcp` over streamable HTTP. It
ships **no credentials**, and it should stay that way: the server implements the
MCP authorization spec — protected-resource metadata, OAuth 2.1 against
`https://api.ukumi.ai`, dynamic client registration, PKCE `S256` — so both hosts
run the login themselves on first use. Never add an API key or token field here.

For a dev server, override locally rather than editing the committed file:

```bash
claude mcp add --transport http montage-dev https://dev-mcp.montage.app/mcp
```

## ChatGPT wiring (one manual step, not yet done)

ChatGPT resolves a remote MCP server through a *registered connection*, not the
URL alone. `.app.json` currently holds a placeholder, and
`.codex-plugin/plugin.json` deliberately does **not** reference it — an
unreferenced placeholder is inert, a referenced one breaks the install.

To finish it:

1. ChatGPT → Settings → Security and login → turn on **Developer mode**.
2. [chatgpt.com/plugins](https://chatgpt.com/plugins) → **+** → enter
   `https://mcp.montage.app/mcp`.
3. Copy the connection ID from the resulting URL. Strip the leading `plugin_`:
   the URL shows `plugin_asdk_app_…`, and the file wants `asdk_app_…`.
4. Put it in `.app.json`, then add `"apps": "./.app.json"` to
   `.codex-plugin/plugin.json`.

## Test locally

Validate the manifests — this catches schema mistakes before anyone installs:

```bash
claude plugin validate .
claude plugin validate .claude-plugin/marketplace.json
```

Claude Code, against your working copy:

```bash
claude plugin marketplace add .
claude plugin install montage@montage
```

Codex / ChatGPT desktop: the repo marketplace at
`.agents/plugins/marketplace.json` is read automatically when the repo is your
working directory. Restart the ChatGPT desktop app, then find **Montage** in the
Plugins Directory. Or from the CLI:

```bash
codex plugin marketplace add ./
codex plugin list
```

Skill edits take effect immediately in a running session. Changes to manifests,
`agents/`, `.mcp.json`, or `hooks/` do not — run `/reload-plugins` or restart.

## Releasing

Bump `version` in **both** `.claude-plugin/plugin.json` and
`.codex-plugin/plugin.json` in the same commit. They are independent files and
nothing cross-checks them.

Claude Code treats the version as the update trigger: leave it unchanged and
installed users never see the new code, because it falls back to pinning the git
commit SHA. `claude plugin tag` validates that the manifest and the marketplace
entry agree before tagging a release.
