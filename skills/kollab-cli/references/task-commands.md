# task commands

Task commands cover WebUI-equivalent conversation creation, high-fidelity chat turns, history, messages, prompt management, moving, sharing, and task collaboration permissions.

## Basic Operations

```bash
kollab task list --page 1 --page-size 20
kollab task list --keyword "session storage" --start-date <YYYY-MM-DD> --end-date <YYYY-MM-DD>
kollab task create --project-id <project-id> --model-tier auto
kollab task create --model-tier auto --byok-credential-id <credential-uuid>
kollab task create --model-tier auto --composer-mode slides
kollab task create --model-tier auto --question "Draft an outline" --project-id <project-id>
kollab task info --conversation-id <conversation-id>
kollab task rename --conversation-id <conversation-id> --title "New Title"
kollab task delete --conversation-id <conversation-id>
```

## Chat Execution

```bash
kollab task ask --question "Summarize this project" --project-id <project-id> --model-tier auto
kollab task ask --model-tier auto --conversation-id <conversation-id> --question "Continue"
kollab task ask --conversation-id <conversation-id> --question "Continue" --model-tier auto --byok-credential-id <credential-uuid>
kollab task ask --conversation-id <conversation-id> --question "Create an eight-slide launch deck" --model-tier auto --composer-mode slides --composer-mode-options '{"format":"polished"}'
kollab task ask --model-tier auto --conversation-id <conversation-id> --question-file /tmp/large-input.txt
kollab task ask --model-tier auto --conversation-id <conversation-id> --question "Compare these two files" --file-ids <project-file-id-1>,<project-file-id-2>
kollab task ask --model-tier auto --conversation-id <conversation-id> --question "Only use these connectors" --mcp-server-ids <mcp-id-1>,<mcp-id-2>
kollab task ask --model-tier auto --conversation-id <conversation-id> --question "Expand on this" --quoted-message-id <message-id>
kollab task ask --model-tier auto --question "Make a similar version" --remix-share-token <32-char-hex-token>
kollab task ask --model-tier auto --question "Make a similar version" --remix-share-token <32-char-hex-token> --remix-source-task-title "Source Task"
kollab task answer-question --model-tier auto --conversation-id <conversation-id> --tool-use-id <tool-use-id> --answers '{"approval":"allow"}'
kollab task messages --conversation-id <conversation-id>
kollab task messages --conversation-id <conversation-id> --after <message-id>
kollab task history --conversation-id <conversation-id> --mode full
kollab task generation-status --conversation-id <conversation-id>
kollab task prompt --conversation-id <conversation-id>
kollab task stop --conversation-id <conversation-id>
```

## Relay Reproduction (debug)

`task relay-test` reproduces a same-conversation relay — a follow-up sent while the prior turn is still generating — without a browser. It starts a turn, waits until `generation_status=generating`, fires one or more relays mid-generation (retrying `RELAY_NOT_ACTIVE` like the frontend), bounds the wait, then checks that generation settles, accepted user messages remain, and each accepted follow-up has a corresponding non-empty assistant text answer or an explicit interruption. An HTTP 202 ack alone is not an answer.

```bash
# Single relay — verify the turn settles instead of hanging in `generating`
kollab task relay-test --model-tier auto \
  --question "画一张赛博朋克城市夜景" \
  --relay-question "换成白天" \
  --relay-after 5 --max-wait 180

# Rapid burst — verify no follow-up is ever bounced back to the user
kollab task relay-test --model-tier auto \
  --question "画一张赛博朋克城市夜景" --relay-question "换个风格" \
  --relay-count 5 --relay-interval 2 --relay-after 4
```

Flags beyond the two required prompts: `--relay-after` (delay after generation starts), `--max-wait` (settle budget before declaring stuck), `--generating-wait` (budget for `generating` to appear), `--relay-count` / `--relay-interval` (burst size and spacing), plus `--conversation-id` (reuse), `--project-id`, `--model-tier`.

Report and exit codes:

- `stuck: true` → `generation_status` is still `generating` after `--max-wait`. Capture the transcript JSONL for that `conversationId` immediately (GC'd after hours). Exit code `2`.
- `bouncedRelays > 0` → an accepted follow-up is missing from canonical user messages. Exit code `3`. For a fresh conversation, `userMessageCount` should equal `1 + relayCount`; reused conversations include prior users.
- `unansweredRelays > 0` → an accepted follow-up has neither a corresponding non-empty text answer nor an explicit interrupted assistant. This includes missing user messages and is silent loss, even when generation is idle. Exit code `4`.
- `explicitlyFailedRelays > 0` → a corresponding assistant has `metadata.status: "interrupted"` or `metadata.interrupted: true`, including a partial interrupted answer. This is an explicit failure, not silent loss. Exit code `5` unless another successful corresponding answer exists.
- Per-entry `relays[]` fields: `userMessageId` from the ack, `presentInFinal`, `answered`, `explicitlyFailed`, plus `accepted`, `index`, uniquely marked `question`, `attempts`, and `lastError`.
- Match the ack user ID (or this run's unique question marker when absent), then require the assistant to follow that user in canonical order and match `parent_id`. `metadata.coveredUserMessageIds` only supplements shared answers for users at or before that parent, with the parent itself before the answer and included in the list; stale coverage cannot claim later users after a split. Legacy unsplit `relaySplit.assistantMessageId/partIndex` only counts text after that block boundary; pre-existing A text, thinking and tool-only blocks do not count.
- Exit precedence: stuck `2`, bounced `3`, unanswered `4`, explicit interruption `5`, rejected/unaccepted `1`, otherwise `0`. The complete report is printed even on diagnostic failure; `ok` is false for these failures. `hint` includes unanswered and explicit-failure counts.
- `--max-wait` also bounds canonical readback after the main SSE ends, because assistant-final persistence can lag. A pending answer at this deadline is reported as unanswered; increase the budget for slow tasks. A history read failure is an error, never an empty successful history.
- Use a first `--question` that triggers a long tool (image generation) so relays land mid-generation; the timing window is non-deterministic, so rerun a few times and raise `--relay-after` if relays are rejected before generation starts.

Notes:

- In a generating conversation, `task ask --conversation-id ...` waits in the server's conversation queue and runs sequentially after the current turn; it does not interrupt that turn. Keep the asking process connected. This ordinary `/chat/stream` queue is distinct from `relay-test`'s `/chat/relay` path, which queues input on the active SDK client with `priority=later`.
- `task stop` requests interruption of the current turn. Ordinary waiting `task ask` calls can proceed after the current lock releases. The intended product rule is “stop current only; keep queued messages”, but the current relay fallback/requeue path can mark queued drafts interrupted/cancelled on Stop; continuing every accepted relay is not yet guaranteed. Read canonical status rather than assuming Stop answered or preserved every queued relay.
- `task create` uses the same `/qa/conversation` path and payload shape as WebUI; `--project-id` becomes `parentId`
- `task create --composer-mode <mode>` maps to WebUI's persisted `composerMode`. `task ask`
  maps `--composer-mode` / `--composer-mode-options` to request-level `composer_mode` /
  `composer_mode_options`. Options must be a JSON object whose values are non-empty strings;
  mode and option names remain server-owned product contracts rather than a CLI allowlist.
- `task ask` and `task answer-question` both print one high-fidelity JSON bundle containing `request`, `stream.events`, `summary`, `persisted`, plus top-level `answer` / `text`
- `task ask --question-file /tmp/input.txt` reads prompt text from a local file. If `--question` or `--question-file` exceeds the safe direct JSON request budget, the CLI uploads the original text as a `text/plain` attachment, sends `Please use the attached file.`, and forwards both `attachments` and `uploaded_file_keys` for managed runtime workspace restore.
- `timing.streamDurationMs` is the Web-like SSE duration; `timing.persistedDurationMs` is CLI-only settle/readback. Internal `artifacts_synced` events are still counted in `summary.artifactEventCount`, but only `summary.visibleArtifactEventCount` makes CLI wait for non-empty artifact truth. This keeps pure-text turns fast while artifact-producing turns still wait for files.
- `task ask --remix-share-token ...` forwards `remix_source` through the same server-side remix pipeline as the Web share entry, including workspace reference import and first-turn runtime guidance
- `task answer-question` expects `--answers` to be a JSON object whose values are strings or arrays of strings
- `--file-ids` (project file block IDs) and `--mcp-server-ids` accept comma-separated values or a JSON array; both narrow the turn to those resources instead of the conversation defaults
- `--quoted-message-id` attaches an existing message as the quoted context of the new turn, matching WebUI quote-reply
- `task ask` and `task answer-question` both accept `--agent-id` to override the agent used for that single turn
- `messages` uses the canonical message API, including body text, reasoning, attachments, and tool-call summaries
- `history --mode full` is the right choice when you need a richer paginated history view

## Prompt and Context

```bash
kollab task set-prompt --conversation-id <conversation-id> --custom-prompt "You are a strict reviewer"
kollab task set-prompt --conversation-id <conversation-id> --clear
kollab task clear --conversation-id <conversation-id>
```

## Search, Share, and Project Binding

```bash
kollab task search --keyword "session storage"
kollab task share --conversation-id <conversation-id>
kollab task share-status --conversation-id <conversation-id>
kollab task shared-view --share-token <32-char-hex-token>
kollab task move --conversation-id <conversation-id> --project-id <project-id>
kollab task move --conversation-id <conversation-id> --detach
```

Notes:

- `shared-view` is a public share endpoint and does not depend on private task permissions
- `share-token` must be a 32-character hexadecimal string
- For share-remix flows, first read any imported local references such as `./remix-source/CLAUDE.md`, then run `shared-view` once to recover the source transcript recipe without assuming private workspace access

## Task References (Web Links)

```bash
kollab task create-link --conversation-id <conversation-id> --title "<title>" --url "<url>"
kollab task create-link --title "<title>" --url "<url>" --description "Why this source matters"
```

Notes:

- Saves a web link as a task-scoped reference so it becomes queryable alongside the task's other sources. This is the task-level counterpart of `project create-link`; the server derives the parent block and space.
- `--title` and `--url` are required; `--conversation-id` defaults to `$CURRENT_CONVERSATION_ID`.

## Collaboration Permissions

```bash
kollab task access --conversation-id <conversation-id>
kollab task invite --conversation-id <conversation-id> --email user@example.com
kollab task invite-link --conversation-id <conversation-id>
kollab task remove-member --conversation-id <conversation-id> --user-id <user-id>
```

## File Upload (Chat Attachments)

```bash
kollab task upload --file /path/to/report.pdf --conversation-id <conversation-id>

# When running inside a managed runtime where CURRENT_CONVERSATION_ID is set:
kollab task upload --file /path/to/report.pdf
```

Output includes:

- `fileKey` — the storage key; pass to `--uploaded-file-keys`
- `attachments` — ready-to-paste JSON array for `--attachments`; entries include the standard `url`, `mediaType`, and `filename` fields plus snake_case metadata for scripts
- `taskAskExample` — a complete `kollab task ask` command with both flags pre-filled

Typical AI flow:

```bash
# 1. Upload the file and capture the output
UPLOAD=$(kollab task upload --file /tmp/data.csv)

# 2. Extract the key and attachments JSON
FILE_KEY=$(echo "$UPLOAD" | jq -r '.data.fileKey')
ATTACHMENTS=$(echo "$UPLOAD" | jq -c '.data.attachments')

# 3. Run the task ask with the uploaded file
kollab task ask --model-tier auto \
  --conversation-id <conversation-id> \
  --question "Summarise this data" \
  --uploaded-file-keys "[\"$FILE_KEY\"]" \
  --attachments "$ATTACHMENTS"
```

Notes:

- Maximum file size: **500 MB**.
- `--conversation-id` defaults to `$CURRENT_CONVERSATION_ID` when running inside a managed runtime.
- The command POSTs to `/skills/chat/upload-url` to get a presigned URL, then PUTs bytes directly to storage.
- `fileKey` is the storage object key; `attachments` is the JSON array that `task ask --attachments` expects.

## Message and Conversation Management

```bash
# Set a star rating on an assistant message (1=worst, 5=best). Omit --rating to clear.
kollab task message-feedback --message-id <uuid> --rating 5
kollab task message-feedback --message-id <uuid> --rating 3 --comment "Slightly off topic"
kollab task message-feedback --message-id <uuid>  # clears existing feedback

# Delete a message and ALL its descendants (cascade, irreversible). Requires --confirm.
kollab task delete-message --message-id <uuid> --confirm

# Check background background task context window usage for the current conversation.
# Returns { status: 'unavailable' } when no active background task session exists yet.
kollab task context-usage
kollab task context-usage --conversation-id <uuid>

# Persist a model selection on a conversation.
# Use auto by default; concrete model ids are validated server-side, with selection validation also applied locally.
kollab task set-model-tier --tier auto
kollab task set-model-tier --tier auto --byok-credential-id <credential-uuid>
kollab task set-model-tier --tier auto --conversation-id <uuid>
```

Notes:

- `message-feedback --rating` accepts integers 1-5. Omitting `--rating` (or no flag) sends `value: null`, which clears any existing feedback for that (message, user) pair.
- `delete-message` cascades: the server walks `parent_message_id` recursively and deletes the target message AND all its descendants (the assistant reply, any tool-result messages, and all branches below). This cannot be undone.
- `context-usage` reads the background task session; it does not report the interactive chat context. Without a bound background task session it returns `{ status: 'unavailable' }`, which is not an error.
- `set-model-tier` persists the model selection to the conversation row, overriding the space default for that specific conversation. The change takes effect on the next chat turn. `--conversation-id` defaults to `$CURRENT_CONVERSATION_ID`.
- `--byok-credential-id` is stored separately from `modelTier`; omit it on `set-model-tier` to clear BYOK and select the platform model.
- `--model-tier` / `--tier` deliberately accept arbitrary concrete model-id strings and rely on the server's current model routing table for validation. A model-list update should change server routing/validation and UI/docs, not the CLI parser.

## Long-task Polling


```bash
# List all active and recent long tasks for a conversation.
kollab task long-task list --conversation-id <conversation-id>

# Filter list by job kind: video | tts | asr | all (default: all; filtering is client-side)
kollab task long-task list --conversation-id <conversation-id> --kind tts
kollab task long-task list --conversation-id <conversation-id> --kind video --limit 10

# Check one specific long task by job id.
kollab task long-task status --conversation-id <conversation-id> --job-id <job-id>

# Block until the job reaches a terminal state or a timeout expires.
kollab task long-task watch \
  --conversation-id <conversation-id> \
  --job-id <job-id> \
  --timeout 180 \
  --interval 5

# Hide a long-task history record from the conversation view (does NOT cancel the job
# or delete artifacts; only removes the history entry). Requires --confirm.
kollab task long-task hide \
  --conversation-id <conversation-id> \
  --job-id <job-id> \
  --confirm
```

AI flow inside a doubao-tts turn:

```bash
# 1. doubao-tts generate "Hello"  → returns {"job_id": "<id>", ...}
# 2. kollab task long-task watch --conversation-id $CURRENT_CONVERSATION_ID --job-id <id> --timeout 180 --interval 5
# 3. On success the watch result includes audio_url / s3_key for diagnostics,
#    but the MP3 is registered as a task artifact automatically. Do not paste
#    the signed URL into prose. On timeout the watch returns watch_result:
#    "timeout" — tell the user the task is still in progress and end the turn
#    (the long-task callback delivers the final result through the task
#    notification stream).
```

Notes:

- `long-task list --kind` accepts `video | tts | asr | all` (default: `all`). The server returns all types; filtering is applied client-side by the CLI. Use `--limit` (default 20) to control how many jobs to show.
- `long-task watch` polls every `--interval` seconds (default 5) until `--timeout` seconds elapse (default 300) or a terminal status (`succeeded`, `failed`, `cancelled`) is observed. `--interval` can be fractional (e.g. `--interval 2.5`).
- The `watch_result` field in the output is `"succeeded"`, `"failed"`, or `"timeout"`.
- On success, the `result` field contains provider-specific output (e.g. `audio_url`, `s3_key` for TTS; `text`, `transcript_json_key` for ASR).
- Use `watch` for polling so a single tool call covers the wait and the agent turn keeps its tool-call budget.
- `long-task hide` writes an storage tombstone so the record no longer appears in history lists. It does NOT cancel the provider job, does NOT delete artifacts, and does NOT affect running jobs. Use it to clean up stale or failed records from the conversation view.
