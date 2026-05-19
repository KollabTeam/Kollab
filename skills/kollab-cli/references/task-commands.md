# task commands

Task commands cover WebUI-equivalent conversation creation, high-fidelity chat turns, history, messages, prompt management, moving, sharing, and task collaboration permissions.

## Basic Operations

```bash
kollab task list --page 1 --page-size 20
kollab task create --project-id <project-id> --model-tier lite
kollab task create --question "Draft an outline" --project-id <project-id>
kollab task info --conversation-id <conversation-id>
kollab task rename --conversation-id <conversation-id> --title "New Title"
kollab task delete --conversation-id <conversation-id>
```

## Chat Execution

```bash
kollab task ask --question "Summarize this project" --project-id <project-id> --model-tier lite
kollab task ask --conversation-id <conversation-id> --question "Continue"
kollab task ask --question "Make a similar version" --remix-share-token <32-char-hex-token>
kollab task ask --question "Make a similar version" --remix-share-token <32-char-hex-token> --remix-source-task-title "Source Task"
kollab task answer-question --conversation-id <conversation-id> --tool-use-id <tool-use-id> --answers '{"approval":"allow"}'
kollab task messages --conversation-id <conversation-id>
kollab task messages --conversation-id <conversation-id> --after <message-id>
kollab task history --conversation-id <conversation-id> --mode full
kollab task generation-status --conversation-id <conversation-id>
kollab task prompt --conversation-id <conversation-id>
kollab task stop --conversation-id <conversation-id>
```

Notes:

- `task create` uses the same `/qa/conversation` path and payload shape as WebUI; `--project-id` becomes `parentId`
- `task ask` and `task answer-question` both print one high-fidelity JSON bundle containing `request`, `stream.events`, `summary`, `persisted`, plus top-level `answer` / `text`
- `task ask --remix-share-token ...` forwards `remix_source` through the same server-side remix pipeline as the Web share entry, including workspace reference import and first-turn runtime guidance
- `task answer-question` expects `--answers` to be a JSON object whose values are strings or arrays of strings
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

## Collaboration Permissions

```bash
kollab task access --conversation-id <conversation-id>
kollab task invite --conversation-id <conversation-id> --email user@example.com
kollab task invite-link --conversation-id <conversation-id>
kollab task remove-member --conversation-id <conversation-id> --user-id <user-id>
```

## Long-task Polling

Long tasks (video generation, Doubao TTS, Doubao ASR) are queued by calling the provider skill (e.g. `/doubao-tts generate ...`), which returns a `job_id` immediately. The agent then polls with:

```bash
# List all active and recent long tasks for a conversation.
kollab task long-task list --conversation-id <conversation-id>

# Check one specific long task by job id.
kollab task long-task status --conversation-id <conversation-id> --job-id <job-id>

# Block until the job reaches a terminal state or a timeout expires.
kollab task long-task watch \
  --conversation-id <conversation-id> \
  --job-id <job-id> \
  --timeout 180 \
  --interval 5
```

AI flow inside a doubao-tts turn:

```bash
# 1. doubao-tts generate "Hello"  → returns {"job_id": "<id>", ...}
# 2. kollab task long-task watch --conversation-id $CURRENT_CONVERSATION_ID --job-id <id> --timeout 180 --interval 5
# 3. On success the watch result includes audio_url / s3_key; embed in the
#    response. On timeout the watch returns watch_result: "timeout" — tell
#    the user the task is still in progress and end the turn (the long-task
#    callback delivers the final result through the task notification stream).
```

Notes:

- `long-task watch` polls every `--interval` seconds until `--timeout` seconds elapse or a terminal status (`succeeded`, `failed`, `cancelled`) is observed.
- The `watch_result` field in the output is `"succeeded"`, `"failed"`, or `"timeout"`.
- On success, the `result` field contains provider-specific output (e.g. `audio_url`, `s3_key` for TTS; `text`, `transcript_json_key` for ASR).
- Use `watch` for polling so a single tool call covers the wait and the agent turn keeps its tool-call budget.
- Audio routes through `/doubao-tts` (text→speech) and `/doubao-asr` (speech→text), both backed by Volcano Doubao via skills-server.
