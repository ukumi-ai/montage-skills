# montage-skills

Agent skills for [Montage](https://ukumi.ai) — video imports, clip creation and recorded-knowledge research via the Montage MCP server.

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
| [repurpose-recordings](skills/repurpose-recordings/SKILL.md) | Turn recordings into evidence-backed summaries, posts, case studies or cross-video research in the assistant. |

All skills require the Montage MCP server to be connected.

## Adding a skill

Skills are byte-exact copies of the Montage MCP server’s canonical skills.
Update the server source first, then copy the same bytes into both this repo
and montage-plugin. Internal release CI checks for drift; do not independently
rewrite the copies.

## Release compatibility

Upload v2 is enabled by the server rollout, not by installing a skill. Resume,
quota choices and silent-footage handling require it on the connected server.
Use returned product state and readiness fields; completed transport alone is
not success. This update does not add local uploads, automatic clip finding, HD
export or social posting.
