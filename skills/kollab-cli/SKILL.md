---
name: kollab-cli
description: |
  Manage Kollab workspace resources through the `kollab` CLI. Use this skill whenever an agent needs auth, spaces, projects, tasks, skills, artifacts, timers, bots, MCP servers, memory, feedback, prompt-gallery search, or any workspace resource operation. Use the kollab CLI for any capability it covers. For extracting content from external URLs (podcasts, videos, articles, social posts), route through `/kollab-social`, which orchestrates the Spotify / Apple Podcasts / Xiaoyuzhou / YouTube / RSS / TikTok / etc. resolvers and the runtime-managed provider bridge.
allowed-tools: Bash, Read
license: MIT
metadata:
  author: KollabTeam
  version: "0.1.4"
---

# kollab-cli

`kollab` is the standard command-line entrypoint for Kollab workspace resources. Runtime prompts should mention capabilities, not exact syntax; read this skill and the relevant reference before using unfamiliar commands.

## Skill Boundary

This skill is for **workspace resource operations only** — auth, spaces, projects, tasks, skills, artifacts, timers, bots, MCP servers, memory, feedback, prompt-gallery search, and the workspace mutation commands documented below.

For **extracting external content** (any URL the user pastes — podcast episodes, video transcripts, social posts, ordinary webpages, RSS feeds, public audio files), use `/kollab-social`. That skill auto-routes Spotify / Apple Podcasts / Xiaoyuzhou (小宇宙) / YouTube / TikTok / Douyin / Instagram / X / RSS / direct audio URLs through the appropriate resolver and returns transcripts plus show/episode metadata.

## Usage Rules

1. Run `kollab --help` or `<group> --help` before using a command shape you do not know.
2. For mutations, inspect first with `get`, `list`, `history`, or an equivalent read command.
3. If the CLI covers the capability, use the CLI command for it.
4. Respect model-tier plan limits: free spaces are limited to `lite`; `pro` and `max` require a higher plan.
5. In Agent/Bot runtime, obey `allowed_cli_commands` from the prompt. A 403 usually means the command group is not authorized.
6. `feedback` and `imagine` are always available regardless of `allowed_cli_commands`.
7. For share-remix from CLI, use `kollab task ask --remix-share-token <token>` so the request enters the same server-side `remix_source` pipeline as WebUI. Inside runtime, inspect `./remix-source/` first, then call `kollab task shared-view --share-token <token>` once to read the public transcript recipe.

## Command Groups

| Group | Purpose |
| --- | --- |
| (top-level auth) | `kollab login`, `kollab logout`, `kollab status` — top-level commands, there is no `auth` group prefix |
| knowledge | ask/search project knowledge and read documents |
| space | space info, members, invites, permission groups, usage, token usage, billing, top-ups |
| project | project CRUD, document search/read, block operations, collaboration permissions |
| task | conversation CRUD, chat turns, questionnaire continuation, history, messages, prompts, share, move, delete, stop, collaboration permissions, long-task polling |
| skill | workspace skill listing, marketplace search (`skill search`), session-scoped install (`skill try`), space-wide official install, and GitHub skill import (`skill install`) |
| artifact | conversation artifacts, workspace-sync files, artifact actions |
| timer | scheduled tasks, model overrides, MCP/project/source-conversation binding |
| bot | bot management, knowledge scope, CLI permissions, platform config, model tier |
| mcp | workspace MCP server configuration |
| memory | space memory file management |
| feedback | search, report, or comment on user-facing issues in `KollabTeam/Kollab` |
| imagine | search public Kollab Prompt Gallery prompts and list gallery tags |
| workspace | unified namespace re-exporting space/project/task/skill/artifact/timer/bot/mcp/memory |

There is no `marketplace` command. Use `kollab skill search` to search the official skill catalog (keyword-paginated; only sees system skills, not user-private custom skills), `kollab skill try` for session-scoped install (workspace only, no space ACL change), `kollab skill install --id <skill-id>` for space-wide official install, and `kollab skill install --github-url <url>` to import a GitHub skill into the current space. Use `kollab imagine` for prompt/creative-reference discovery.

## Auth and Environment

Preferred environment variables:

- `KOLLAB_API_URL`: site base URL, for example `https://kollab.im` without `/api`.
- `KOLLAB_TOKEN`.
- `KOLLAB_API_TOKEN`.
- `KOLLAB_SPACE_ID`.
- `CURRENT_CONVERSATION_ID`.

Common commands:

```bash
kollab status
kollab login
kollab login --token "$KOLLAB_TOKEN"
kollab logout
```

Notes:

- `kollab login` starts a browser flow, reuses the web account session, and asks the user to choose a space.
- `kollab login --token ...` is the fallback for scripts or pre-issued workspace tokens.
- Some tokens lack a default space; set `KOLLAB_SPACE_ID` when resource commands need it.
- `timer create/update` binds `CURRENT_CONVERSATION_ID` when `--source-conversation-id` is omitted and `--standalone` is not set.
- `timer create` inherits the bound conversation's `modelTier` unless `--model-override` is explicit.

## Authentication Failures

Treat these two messages differently:

**`Not logged in. Run kollab login ...`**

- No token env var and no `~/.kollab/credentials.json`.
- Typical local developer-shell state.
- Fix with `kollab login` or intentional `kollab login --token <token>`.

**`KOLLAB_API_TOKEN is set but empty...`**

- The AgentCore host attempted workspace-token injection but produced an empty value.
- Typical runtime/host failure signal.
- Do not run `kollab login`; it cannot fix host injection.
- Ask the platform owner to inspect skills-server logs for `[WORKSPACE-TOKEN] CLI token not injected`. That log names the blocker: missing `space_id`, missing `X-User-Id`, missing `WORKSPACE_SERVICE_API_KEY`, or upstream 4xx/5xx from `/api/auth/workspace-token`.

When either error appears mid-task, stop retrying CLI commands, report the exact variant, and continue with local-only alternatives when possible.

## Quick Examples

```bash
kollab space usage
kollab space billing-subscription
kollab space billing-catalog
kollab space invite

kollab knowledge ask --question "Summarize this project" --project-id <project-id>
kollab knowledge search --question "Summarize this project" --top-k 8
kollab knowledge list-documents --project-id <project-id> --query PRD
kollab knowledge get-document-content --document-id <file-uuid>

kollab workspace project list
kollab project access --id <project-id>

kollab workspace task create --project-id <project-id> --model-tier lite
kollab task ask --conversation-id <conversation-id> --question "Summarize this project"
kollab task ask --question "Make a similar version" --remix-share-token <32-char-hex-token> --remix-source-task-title "Source Task"
kollab task answer-question --conversation-id <conversation-id> --tool-use-id <tool-use-id> --answers '{"approval":"allow"}'
kollab task history --conversation-id <conversation-id>
kollab task invite --conversation-id <conversation-id> --email user@example.com

kollab bot create --name "Ops Bot" --platform feishu --model-tier pro
kollab bot update --id <bot-id> --model-tier default

# After a video/TTS/ASR job is submitted, poll it to completion inside the same turn:
kollab task long-task list --conversation-id $CURRENT_CONVERSATION_ID
kollab task long-task status --conversation-id $CURRENT_CONVERSATION_ID --job-id <job-id>
kollab task long-task watch --conversation-id $CURRENT_CONVERSATION_ID --job-id <job-id> --timeout 180 --interval 5
```

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

## Model Tier Notes

Model selection is supported by:

- `bot create --model-tier <lite|pro|max|default>`
- `bot update --model-tier <lite|pro|max|default>`
- `task create --model-tier <lite|pro|max>`
- `task ask --model-tier <lite|pro|max>`
- `task answer-question --model-tier <lite|pro|max>`
- `timer create --model-override <model>`
- `timer update --model-override <model>`

Notes:

- `bot --model-tier default` clears the override and falls back to the space default.
- `task create` uses the same `/qa/conversation` payload shape as WebUI; `--project-id` maps to `parentId`.
- `task ask` and `task answer-question` print high-fidelity JSON with request, raw stream events, compact summary, canonical messages, and artifacts.

## References

Read only the relevant file:

- `references/auth-commands.md`
- `references/knowledge-commands.md`
- `references/space-commands.md`
- `references/project-commands.md`
- `references/task-commands.md`
- `references/skill-commands.md`
- `references/artifact-commands.md`
- `references/timer-commands.md`
- `references/bot-commands.md`
- `references/mcp-commands.md`
- `references/memory-commands.md`
- `references/feedback-commands.md`
- `references/imagine-commands.md`
- `references/workspace-commands.md`
