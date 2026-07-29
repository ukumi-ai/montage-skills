# montage-skills

Agent skills for [Montage](https://ukumi.ai) — video ingestion and clip creation via the Montage MCP server.

## Install

```bash
npx skills add ukumi-ai/montage-skills
```

Or find them on [skills.sh](https://skills.sh).

## Skills

| Skill | What it does |
|-------|--------------|
| [ingest-video](skills/ingest-video/SKILL.md) | Ingest a video into Montage from a Google Drive or public URL and track the pipeline to completion. |
| [create-moment](skills/create-moment/SKILL.md) | Create an editable moment (clip) from an ingested video and deliver an editor link. |

Both skills require the Montage MCP server to be connected.

## Adding a skill

Drop a new directory under `skills/<name>/` containing a `SKILL.md` with `name` and `description` frontmatter, then push to `main`.
