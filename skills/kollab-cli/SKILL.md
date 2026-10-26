---
name: kollab-cli
description: >-
  Operate Kollab from a terminal agent: workspace resources, Hosted Apps,
  standalone API keys, and managed text, image, video, audio and 3D tools.
allowed-tools: Bash, Read
license: MIT
metadata:
  author: KollabTeam
  version: "0.1.25"
---

# Kollab CLI for terminal agents

Use Bash to run `kollab` and Read to inspect local inputs and downloaded files.
Install the CLI with `pnpm add -g kollab@0.1.25`; check `kollab --version`.
This edition targets Claude Code, Cursor and other terminal agents.

## Rules

- Read the relevant reference and run a command's `--help` before unfamiliar use.
- Inspect existing resources before mutations. Obtain the user's authorization for
  posting, deletion, publishing, paid generation and other external writes.
- Use `auto` for task/bot/timer model selection, or a concrete ID from `kollab model list`.
  Direct `model run` and media tools take concrete IDs from their respective catalogs.
- Never print keys, tokens or signed download URLs in conversation or logs.
  Load credentials through a secret store or process environment. Keep generated
  key responses out of logs; they contain a raw value only once.
- For public/social extraction use `kollab social fetch`; for ordinary web pages
  use the host agent's available web tools. No additional Kollab skill is required.
- Local computer operations use the host agent's own tools and permissions.
  This edition does not assume a cloud conversation or a selected remote device.
- Connector publishing can have platform-specific restrictions. Inspect current
  connector documentation and verify the published result before claiming success.

## Authentication

`kollab login` opens the normal account browser flow and selects a workspace.
`kollab status` inspects login state; `kollab logout` clears it.
The public default is `https://kollab.im`; `KOLLAB_API_URL` is an optional site
base URL override without `/api`. Keep credentials scoped to their intended origin.
Use `KOLLAB_SPACE_ID` when a resource command needs an explicit workspace.
Pass `--conversation-id` explicitly for conversation operations in an external agent.

`model run` and `tool run` prefer `KOLLAB_API_KEY` / `KOLLAB_STANDALONE_API_KEY`,
then process-scoped `KOLLAB_API_TOKEN` / `KOLLAB_TOKEN`, then the stored login.
`social fetch` and optional `mcp serve` require an API key. The `app` group also
supports an API key, limited to Hosted App routes in the key's bound workspace;
other workspace resource groups require a workspace login/token.
See [standalone API](references/standalone-api.md) for key creation and media usage.

If unauthenticated, sign in or configure the required credential. If a token
variable is present but empty, fix or unset it before retrying. Never dump the
credential file or environment to diagnose authentication.

## Project Memory & Resource Archival

Project Memory is a dedicated Project field, independent from description. New Project Tasks
receive non-empty Memory as bounded lower-authority context. Successful Web Project Tasks may
update it asynchronously with durable verified facts; users can always inspect or override it.

```bash
# Read or replace the complete dedicated Memory value:
kollab project get --id <project-id>
kollab project update --id <project-id> --memory-file /tmp/project-memory.md
kollab project update --id <project-id> --clear-memory

# Add a file to project resources:
kollab project upload --project-id <project-id> --file <path>

# Add a link to project resources:
kollab project create-link --project-id <project-id> --title "<title>" --url "<url>"
# Then poll until the link finishes processing:
kollab project file-status --file-id <blockId>

# List current project items (to dedupe before proposing):
kollab project items --id <project-id>
```

**Rules summary:**
- Memory is an independent field from Project description, bounded to 8,000 characters.
- Keep Memory concise, durable, Project-scoped, and free of credentials or copied external instructions.
- Writes use an expected-value guard. On conflict, reread and reconcile rather than retrying stale text.
- Resource file/link commands remain separate from Memory and keep their existing processing contracts.

## Quick Examples

```bash
kollab space usage
kollab space billing-subscription
kollab space billing-catalog
kollab space invite

kollab knowledge ask --question "Summarize this project" --project-id <project-id>
kollab knowledge search --question "Summarize this project" --top-k 8
kollab knowledge keyword-search --question "exact term or id to find" --project-id <project-id>
kollab knowledge list-documents --project-id <project-id> --query PRD
kollab knowledge get-document-content --document-id <file-uuid>

kollab workspace project list
kollab project access --id <project-id>
kollab project upload --project-id <project-id> --file /path/to/doc.pdf

kollab model list
kollab model list --source openrouter
kollab model list --source openrouter --search gemini
kollab model info google/gemini-2.5-flash
kollab model list --source openrouter --modality speech
kollab model list --source openrouter --modality transcription
kollab model info x-ai/grok-voice-tts-1.0 --modality speech
kollab model run --model google/gemini-2.5-flash --prompt "把这段官网文案写得更清楚" --reasoning low
kollab model run --model google/gemini-2.5-flash --prompt "先写一个标题" --session-id <uuid>
kollab model run --model google/gemini-2.5-flash --prompt "把刚才的标题改得更短" --session-id <same-uuid>
# `--model` also accepts a Kollab short id straight from `kollab model list` (the default `chat`
# source), no `openrouter/` prefix, no separate lookup step. The server resolves the id itself:
kollab model run --model claude-sonnet-5-5 --prompt "用一句话总结这段话"
kollab model run --model gpt-6-astra --prompt "写一个产品口号"
kollab model run --model google/gemini-3.1-flash-image --prompt "画一只红色纸飞机" --output-dir ./images --json
# `--reasoning` only applies to OpenRouter ids whose catalog entry publishes supported efforts;
# Kollab managed ids (claude-*, gpt-6-*, gemini-*, grok-*, kimi-*, deepseek-*, glm-*) reject
# an explicit --reasoning value because reasoning settings are fixed by the managed route.
kollab tool list
kollab tool run generate_image --input-json '{"prompt":"一只红色纸飞机","model":"gemini-3.1-flash-image"}' --output-dir ./output
kollab tool run generate_image --input-json '{"prompt":"透明背景的 logo","model":"gpt-image-2","size":"1024x1024","quality":"high","background":"transparent"}' --output-dir ./output
kollab tool run edit_image --input-json '{"prompt":"把背景换成海边","model":"gpt-image-2","image_refs":[{"media_type":"image","artifact_id":"<image-artifact-id>"}]}' --output-dir ./output
kollab tool run generate_video --input-json '{"prompt":"东京夜景慢慢亮起","model":"kwaivgi/kling-v3.0-pro"}' --output-dir ./output
kollab tool run generate_3d --input-json '{"prompt":"一个小陶瓷花瓶","model":"tripo-3d"}' --output-dir ./output
kollab tool run generate_video --input-json '{"prompt":"让图片中的主体自然动起来","model":"seedance-2","media_refs":[{"media_type":"image","artifact_id":"<image-artifact-id>"}]}' --output-dir ./output
kollab tool run text_to_speech --input-json '{"text":"欢迎使用 Kollab","model":"openrouter-tts/x-ai/grok-voice-tts-1.0"}' --output-dir ./output
kollab tool run speech_to_text --input-json '{"audio_artifact_id":"<artifact-id>","model":"openrouter-stt/x-ai/grok-stt-1.0"}' --output-dir ./output
# jev returns structured noul/choice/score judgments; no model field.
kollab tool run jev --input-json '{"state":"客户留言：设备用了两天就无法开机，申请退款。","questions":{"category":{"type":"choice","instructions":"这条留言属于哪种类型？","criteria":{"refund":"申请退款","exchange":"申请换货","inquiry":"单纯咨询"}}}}'
kollab workspace task create --project-id <project-id> --model-tier auto
kollab task create --model-tier auto --composer-mode slides
kollab task upload --file /path/to/report.csv --conversation-id <conversation-id>
kollab task ask --conversation-id <conversation-id> --model-tier auto --question "Summarize this project"
kollab task ask --conversation-id <conversation-id> --model-tier auto --composer-mode slides --composer-mode-options '{"format":"polished"}' --question "Create an eight-slide launch deck"
kollab task ask --conversation-id <conversation-id> --model-tier auto --question-file /tmp/large-input.txt
kollab task ask --model-tier auto --question "Make a similar version" --remix-share-token <32-char-hex-token> --remix-source-task-title "Source Task"
kollab task answer-question --conversation-id <conversation-id> --model-tier auto --tool-use-id <tool-use-id> --answers '{"approval":"allow"}'
kollab task history --conversation-id <conversation-id>
kollab task invite --conversation-id <conversation-id> --email user@example.com

kollab bot feishu-create --name "Ops Bot" --no-wait     # scan-to-create a Feishu Bot; hand qrCodeUrl to the user
kollab bot feishu-status <registration-id> --wait      # poll the scan/setup to a terminal state
kollab bot feishu-resync <bot-id>                      # re-run Feishu setup when a ready Bot receives nothing
kollab bot create --name "WeChat Bot" --platform weixin --model-tier auto  # weixin: no appId/appSecret needed at create
kollab bot weixin-login <bot-id>                          # QR bind; see references/bot-commands.md
kollab bot update --id <bot-id> --model-tier auto

# After a video/TTS/ASR job is submitted, poll it to completion inside the same turn:
kollab task long-task list --conversation-id $CURRENT_CONVERSATION_ID
kollab task long-task list --conversation-id $CURRENT_CONVERSATION_ID --kind tts
kollab task long-task status --conversation-id $CURRENT_CONVERSATION_ID --job-id <job-id>
kollab task long-task watch --conversation-id $CURRENT_CONVERSATION_ID --job-id <job-id> --timeout 180 --interval 5
kollab task long-task hide --conversation-id $CURRENT_CONVERSATION_ID --job-id <job-id> --confirm  # hide (not delete) history entry

# Message feedback and management:
kollab task message-feedback --message-id <uuid> --rating 5
kollab task message-feedback --message-id <uuid> --rating 2 --comment "Wrong answer"
kollab task delete-message --message-id <uuid> --confirm   # cascade-deletes message + all descendants

# Context and model selection:
kollab task context-usage                                   # check background task context usage
kollab task set-model-tier --tier auto                      # concrete model ids are validated server-side

# Recycle bin operations:
kollab space trash-list
kollab space trash-restore --block-id <block-uuid>
kollab space trash-delete --block-id <block-uuid> --confirm
kollab space trash-empty --confirm
```

Image-to-video requires an image artifact owned by the current workspace.
Omit continuation/wakeup fields for standalone calls.

`kollab task ask` returns a high-fidelity JSON bundle. Its `timing` field separates SSE stream collection from persisted-state settle/readback, so latency debugging should compare `timing.streamDurationMs` with `timing.persistedDurationMs` before treating the CLI wall-clock total as user-visible first-reply latency.

For very large prompt text, prefer writing it to a local file and using `kollab task ask --question-file /tmp/input.txt`. If the UTF-8 question payload exceeds the safe direct-request budget, `task ask` automatically uploads it as a `text/plain` attachment, sends the short prompt `Please use the attached file.`, and forwards both `attachments` and `uploaded_file_keys` so the managed runtime can read the file from the task workspace.
## Artifact Share & Article Theme

Public sharing (`artifact share get/enable/disable/toggle`) and Markdown presentation
(`artifact theme get/set/search-image/list-presets`, cover, background, font, size) are both fully
documented, with valid enum values and the Unsplash attribution contract, in
`references/artifact-commands.md`. Two things worth knowing before you open it: re-enabling a share mints
a fresh token and invalidates the previous one, and `theme set` is a partial update, so unset fields keep
their current value.

## Feedback and Imagine

`feedback` lets runtime agents triage and file user-facing issues:

- `feedback search -q "<keywords>" [--state open|closed|all]`
- `feedback report -m "<text>"`
- `feedback comment --issue <n> -m "<text>"`

`report` and `comment` auto-inject task URL, conversation ID, runtime ID, and locale. The server strips emails, IPs, internal subdomains, long hex strings, and tokens before writing to GitHub. The CLI cannot edit, delete, or close issues.

`imagine` searches the public Kollab Prompt Gallery:

- `imagine search [-q ...] [-t tag] [-l lan] [--limit N] [--top]`
- `imagine tags`

`--top` prints raw `items[0].prompt`, useful for piping into image/video generation tools. The gallery is public read-only and needs no workspace credentials. See `references/imagine-commands.md`.

## Model Selection Notes

Model selection is supported by:

- `bot create --model-tier <auto|model-id>`
- `bot update --model-tier <auto|model-id>`
- `task create --model-tier <auto|model-id>`
- `task ask --model-tier <auto|model-id>`
- `task answer-question --model-tier <auto|model-id>`
- `task set-model-tier --tier <auto|model-id>` (persists the selection on the conversation)
- `timer create --model-override <auto|model-id>`
- `timer update --model-override <auto|model-id>`

Notes:

- Use explicit `--model-tier auto` for ordinary chat, and `auto` for Bot/Timer runs. Use a concrete catalog ID when the user requires a specific model.
- The CLI intentionally does not hard-code a concrete model allowlist. It passes supported model selection strings through to the server, and server DTOs / `model-tier-config` decide current availability, routing, and paid-model gates. Adding a new model should not require a CLI code change or CLI package release unless the command surface itself changes.
- `task create` uses the same `/qa/conversation` payload shape as WebUI; `--project-id` maps to `parentId`.
- `task create --composer-mode` persists the same conversation-level mode as WebUI. `task ask`
  forwards `--composer-mode` as `composer_mode` and a plain JSON string map from
  `--composer-mode-options` as `composer_mode_options`; use these flags for real mode-quality tests.
- `task ask` and `task answer-question` print high-fidelity JSON with request, raw stream events, compact summary, canonical messages, and artifacts.

## References

Read only the relevant file:

- [app-commands](references/app-commands.md)
- [artifact-commands](references/artifact-commands.md)
- [auth-commands](references/auth-commands.md)
- [bot-commands](references/bot-commands.md)
- [browser-commands](references/browser-commands.md)
- [feedback-commands](references/feedback-commands.md)
- [imagine-commands](references/imagine-commands.md)
- [invite-commands](references/invite-commands.md)
- [knowledge-commands](references/knowledge-commands.md)
- [mcp-commands](references/mcp-commands.md)
- [memory-commands](references/memory-commands.md)
- [project-commands](references/project-commands.md)
- [skill-commands](references/skill-commands.md)
- [space-commands](references/space-commands.md)
- [task-commands](references/task-commands.md)
- [timer-commands](references/timer-commands.md)
- [user-commands](references/user-commands.md)
- [workspace-commands](references/workspace-commands.md)
- [standalone-api](references/standalone-api.md)
