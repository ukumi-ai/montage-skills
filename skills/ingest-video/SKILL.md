---
name: ingest-video
description: Import a video into Montage from Drive or Dropbox, resume a stopped MCP upload, or check its pipeline progress.
---

# Ingest Video from URL

Requires upload_video and get_upload_status. Ask for a Google Drive or Dropbox
file link if none was supplied. Drive must allow “Anyone with the link”.
No YouTube, direct URLs, local files or automatic clip finding in this flow.

1. Call `upload_video(video_url=<url>, idempotency_key=<stable-action-key>,
   project_name=<optional-name>)`. Retain the workflow_id. Reuse the SAME key
   and arguments after interruption; this returns the same run.
2. Poll `get_upload_status(workflow_id)` at most 10 times, honoring
   retry_after_seconds. Report step transitions: creating_records →
   validating_url → checking_limits → downloading → validating_video →
   checking_limits → planning → analyzing → finalizing. Child statuses name
   validation, understanding, corpus, transcription and thumbnail. Older v1
   runs may show routing/audio_correction; new uploads keep source audio unchanged.
3. State outranks transport status. A completed workflow can be blocked or
   failed. Stop polling on ready/blocked/failed; retain the ID if still running.
4. On ready, read get_project and get_video_validation (duration, encoded and
   display dimensions, rotation). Read get_transcript, get_video_analysis and
   get_video_corpus(view="rollup") as needed. Silent v2 uploads can be ready
   with ready_for_clipping=false: no transcript, but visual analysis is usable.
   Clips on silent footage are not available over MCP yet.

## Stops and recovery

Show user_guidance/user_message. V2 advice comes from reason codes, v1 from
legacy provider messages. Never silently retry a rejected file.

- usage_limit_reached: explain allowed_seconds/source duration. With the user's
  choice, `upload_video(project_id=<saved-id>, quota_action="process_part",
  idempotency_key=<NEW-action-key>)` processes the first part that fits;
  process_all rechecks the whole file after capacity changes. Nothing bills
  until analysis completes. V1 cannot resume or process a part.
- analysis_unavailable or processing_failed with retryable=true: retry the same
  project_id with a NEW key. The original File and stored object are reused.
- too_short: under 1 second; choose a longer video.
- no_video_stream: choose a video, not an audio-only file.
- video_failed_checks/invalid_video: re-export MP4 (H.264/AAC).
- link_not_readable/not_public/url_blocked: fix sharing or use another supported link.
- drive_rate_limited: wait or share a fresh copy (legacy advice).
- file_too_large/not_a_video: choose a supported video file (legacy advice).
- download failure without a stored source: fix the link and start a NEW upload
  with a new key. Do not resume a missing/deleted project.
- Thumbnail failure is cosmetic; other failed analysis children fail the run.

Optional v2 clip_format: 9:16 (default), 4:5 or 16:9. A nonlandscape source
requested as 16:9 falls back to its own shape. Inspect facts, route_key,
produced/skipped, format_fallback and reframe_needed before describing results.
There is no people-on-camera validation or automatic source denoising.
