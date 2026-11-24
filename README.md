# montage-skills

Agent skills for [Montage](https://ukumi.ai) covering video ingestion, clip creation, and transcript-based writing and research via the Montage MCP server.

## Install

```bash
npx skills add ukumi-ai/montage-skills
```

Or find them on [skills.sh](https://skills.sh).

## Skills

| Skill | What it does |
|-------|--------------|
| [ingest-video](skills/ingest-video/SKILL.md) | Ingest a video into Montage from a Google Drive or Dropbox link and track the pipeline to completion. |
| [create-moment](skills/create-moment/SKILL.md) | Create an editable moment (clip) from an ingested video and deliver an editor link. |
| [repurpose-recordings](skills/repurpose-recordings/SKILL.md) | Use recordings as evidence for posts, case studies, summaries, and cross-video research. |

These skills require a connected Montage MCP server. Available workflows depend on the capabilities the connected server reports.

## Adding a skill

Drop a new directory under `skills/<name>/` containing a `SKILL.md` with `name` and `description` frontmatter, then push to `main`.
