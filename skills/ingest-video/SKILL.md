---
name: ingest-video
description: Use when the user wants to ingest, upload, or import a video into Montage from a Google Drive or public URL, or asks to track/poll an upload_video ingestion workflow through its pipeline steps.
---

# Ingest Video from URL

Ingest a video into Montage from a Google Drive (or any public) URL using the
Montage MCP tools and watch it through every pipeline step.

Argument: the video URL. If none was given, ask for one before doing anything.

Requires the Montage MCP server to be connected with the `upload_video` /
`get_upload_status` tools available (shipped in PR #1298). If the tools are
missing, say so and stop.

## Step 1: Start the ingestion

- Google Drive links must be shared as "Anyone with the link". If the URL is a
  gdrive link, remind the user of this BEFORE starting (a private file fails
  minutes in, during download).
- Call `upload_video(video_url=<url>)`. Optionally pass `project_name` if the
  user provided one. Record the returned `workflow_id`.

## Step 2: Poll until done

Call `get_upload_status(workflow_id)` in a loop, sleeping between calls
(`sleep 30` via Bash; downloading and analyzing are the long steps — do not
poll faster than every 15s).

- Report each **step transition** to the user as it happens, with a plain
  reading of the step: `creating_records` → `validating_url` → `downloading` →
  `validating_video` → `analyzing` (watch `child_workflow_status` for
  understanding / corpus / transcription / thumbnail individually; `thumbnail`
  may be absent on older deployments) → `finalizing`.
- Surface `project_id` / `file_id` as soon as they appear.
- Stop polling when `status` is anything other than `running`.

**Failure handling:**
- `result.success == false` or `status == failed`: report the `message`/
  `error` verbatim. Common causes: gdrive file not public ("Anyone with the
  link"), gdrive rate limit, non-video URL. The project is marked `failed` —
  tell the user they can rerun this command after fixing the share settings.
- A single failed analysis child (see `child_workflow_errors`) is not fatal to
  the others — say which succeeded and which failed. A failed `thumbnail` is
  cosmetic only: it never fails the project or blocks billing.

## Step 3: Verify and summarize

On success:
- `get_project(project_id)` to confirm the project exists and show its name.
- `get_transcript(project_id)` for the transcript.
- Summarize for the user: project name/id, video size, duration if available,
  language, and a 2-3 sentence gist of the content from the transcript.
