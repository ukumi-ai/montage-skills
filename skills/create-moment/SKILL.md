---
name: create-moment
description: Create an editable moment (clip) from an ingested video via the Montage MCP tools and deliver an editor link. Use after video ingestion when the user wants a clip/moment created from their video.
---

# Create a moment from an ingested video

End-to-end flow: read the ingestion outputs, pick clip boundaries, create the
moment, wait for clip preprocessing (downscale + speaker detection), and hand
the user an editor link. All tools are on the Montage MCP server.

## 1. Locate the project/video

- `list_projects` / `get_project` to find the project the user means.
- If the video was just uploaded with `upload_video`, poll
  `get_upload_status(workflow_id)` until `completed` — it returns the
  `project_id` and `file_id`. Ingestion must be complete before creating a
  moment (transcription and video validation results are required).

## 2. Understand the content

- `get_transcript(project_id)` — paginate for full text with timestamps.
- `get_video_corpus(project_id)` for the visual rollup;
  `query_video_corpus_range` to drill into a time range.
- `get_chapters` / `get_binary_labels` for structure and labels.
- `get_moments(project_id)` to see what already exists — don't duplicate.

## 3. Pick segments

Choose one or more `[start, end]` second-pairs from transcript timestamps.
Rules: `0 <= start < end`, within the video duration. Multiple pairs compose
one moment (e.g. a hook plus the payoff). Write a short `title` (and
optionally a `summary`) describing the moment.

## 4. Create the moment

```
create_moment(project_id, title, segments, utterances, summary?, kind?, file_id?)
```

- `utterances` (required for speaker detection): the transcript segments from
  step 2 that overlap the chosen `segments`, as
  `[{"start": <sec>, "end": <sec>, "text": "...", "speaker": "<speaker_id>"}, ...]`
  — map `get_transcript` fields `start_time`→`start`, `end_time`→`end`,
  `speaker_id`→`speaker`, using absolute video seconds
  (`timestamp_format="seconds"`). Cover every chosen segment window; without
  utterances, speaker detection is skipped.
- `kind`: content (default), hook_montage, ad, or sponsor_read.
- `file_id` only when the project has multiple files.
- Requires the `moments:write` scope.

The response contains `moment.id`, `preprocessing.{status, workflow_id}`, and
`editor_url`.

## 5. Wait for preprocessing

- If `preprocessing.status` is `"processing"`: poll
  `get_workflow_status(preprocessing.workflow_id)` every ~30s until `status`
  is `completed` (typically 2–10 minutes). On `failed`/`timed_out`, report it
  — the moment still exists, but the editor may lack the downscaled preview.
- `"ready"` means enrichment already exists; `"skipped"` means preprocessing
  could not run (`preprocessing.reason`: `no_video_url` / `no_input_codec` /
  `no_utterances`) — nothing to poll, tell the user why.

## 6. Deliver

Give the user the `editor_url` from the create_moment response verbatim:

```
https://studio.montage.app/project/{project_id}?view=clipEditing&clipId={moment_id}
```

(`clipId` is the moment id.)
