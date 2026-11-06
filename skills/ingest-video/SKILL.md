---
name: ingest-video
description: Use when the user wants to ingest, upload, or import a video into Montage from a Google Drive or Dropbox link, or asks to track/poll an upload_video ingestion workflow through its pipeline steps.
---

# Ingest Video from URL

Ingest a video into Montage from a Google Drive or Dropbox link using the
Montage MCP tools and watch it through every pipeline step.

Argument: the video URL. If none was given, ask for a Google Drive or Dropbox
link to the video before doing anything. YouTube and direct file URLs are not
accepted.

Requires the Montage MCP server to be connected with the `upload_video` /
`get_upload_status` tools available (shipped in PR #1298). If the tools are
missing, say so and stop.

## Step 1: Start the ingestion

- Google Drive links must be shared as "Anyone with the link". If the URL is a
  gdrive link, remind the user of this BEFORE starting (a private file fails
  minutes in, during download). Dropbox share links work as they are.
- Call `upload_video(video_url=<url>)`. Optionally pass `project_name` if the
  user provided one. Record the returned `workflow_id`.
- Uploads keep source audio unchanged; users choose cleanup per clip at
  export, where Pro access is checked.

## Step 2: Poll until done

Call `get_upload_status(workflow_id)` in a loop, sleeping between calls
(use the host client's waiting mechanism; do not poll faster than every 15s).
If the client cannot keep waiting, retain the workflow ID for resumption and
report that processing is still running; never claim completion.

- Report each **step transition** to the user as it happens, with a plain
  reading of the step: `creating_records` → `validating_url` → `downloading` →
  `validating_video` → `analyzing` (watch `child_workflow_status` for
  understanding / corpus / transcription / thumbnail individually; `thumbnail`
  may be absent on older deployments) → `finalizing`.
- `child_workflow_status.audio_correction` may appear on older, in-flight
  uploads. New uploads never schedule source cleanup.
- Retain `project_id` / `file_id` internally for subsequent calls; use names in user-facing updates.
- Stop polling when `status` is anything other than `running`.

**Failure handling:**
- `result.success == false` or `status == failed`: when the result leads with
  `user_guidance`, tell the user what went wrong and how to fix it from that
  field; otherwise report the `message`/`error` verbatim. Common causes: file
  not shared publicly, Drive rate limit, a link to something that is not a
  video. After the user fixes it, start a new upload with a new
  idempotency_key, even for the same link; the old key returns the failed run.
- `limit_reached: true`: show `user_message`. Follow `assistant_instruction` on when retrying helps.
- A single failed analysis child (see `child_workflow_errors`) is not fatal to
  the others — say which succeeded and which failed. A failed `thumbnail` is
  cosmetic only: it never fails the project or blocks billing.

## Step 3: Verify and summarize

On success:
- `get_project(project_id)` to confirm the project exists and show its name.
- `get_transcript(project_id)` for the transcript.
- `get_video_validation(project_id)` for duration, resolution, fps, codecs and
  whether the footage has usable speech/audio/people.
- `get_video_analysis(project_id)` and `get_video_corpus(project_id,
  view="rollup")` when the user wants the visual read too — these are the same
  outputs the create-moment skill needs.
- Summarize for the user: project name/id, video size, duration if available,
  language, and a 2-3 sentence gist of the content from the transcript.
