# montage-skills

Agent skills and plugins for [Montage](https://ukumi.ai) — video ingestion and
clip creation via the Montage MCP server.

## Install

**Claude Code**

```bash
/plugin marketplace add ukumi-ai/montage-skills
/plugin install montage@montage
```

**ChatGPT / Codex**

```bash
codex plugin marketplace add ukumi-ai/montage-skills
codex plugin add montage
```

Or in the ChatGPT desktop app: open **Plugins**, pick the Montage marketplace,
install from there.

**Skills only, any host**

```bash
npx skills add ukumi-ai/montage-skills
```

Or find them on [skills.sh](https://skills.sh).

Installing the plugin brings the Montage MCP server with it. You sign in to
Montage the first time a tool runs — there is nothing to configure and no API
key to paste.

## What's included

| Skill | What it does |
|-------|--------------|
| [ingest-video](skills/ingest-video/SKILL.md) | Ingest a video into Montage from a Google Drive or public URL and track the pipeline to completion. |
| [create-moment](skills/create-moment/SKILL.md) | Create an editable moment (clip) from an ingested video and deliver an editor link. |

| Agent | What it does |
|-------|--------------|
| [clip-scout](agents/clip-scout.md) | Read-only. Reviews an ingested video and proposes candidate clip segments with timestamps and reasons. Creates nothing. |

## MCP server

`https://mcp.montage.app/mcp` — streamable HTTP, OAuth 2.1 against
`https://api.ukumi.ai`. Scopes: `corpus:read`, `moments:read`, `moments:write`.

Connect it on its own, without the plugin:

```bash
claude mcp add --transport http montage https://mcp.montage.app/mcp
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for the repo layout and how to add a
skill or an agent. Short version: drop a directory under `skills/<name>/` with a
`SKILL.md`, or a file at `agents/<name>.md`. Both manifests already point at
those directories, so nothing else needs editing.
