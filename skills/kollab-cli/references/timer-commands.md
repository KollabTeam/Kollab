# timer commands

Timer commands create, inspect, update, and delete scheduled runs.

## Querying

```bash
kollab timer list
kollab timer list --status <status>
kollab timer history --timer-id <timer-id> --page 1 --page-size 20
```

## Create

```bash
kollab timer create --model-override auto --title "Daily Check" --prompt "Inspect recent errors in the test environment" --schedule-kind cron --cron '0 10 * * 1-5' --timezone Asia/Shanghai --schedule-ui-mode builder --schedule-ui-config '{"days":[1,2,3,4,5],"time":"10:00"}' --byok-credential-id <credential-uuid>
```

Common parameters:

- `--title <text>`
- `--prompt <text>`
- `--schedule-kind <one_time|cron>`
- `--run-at <iso-datetime>`
- `--cron <expr>`
- `--timezone <tz>`
- `--schedule-ui-mode <mode>`
- `--schedule-ui-config <json>`
- `--start-at <iso-datetime>`
- `--end-at <iso-datetime>`
- `--project-id <id>`
- `--source-conversation-id <id>`
- `--create-new-task`: create a new task for each run
- `--append-to-source`: append each run to the bound source conversation
- `--mcp-server-ids <csv-or-json-array>`
- `--knowledge-base-scope <csv-or-json-array>`
- `--agent-id <id>`
- `--model-override <auto-or-model-id>`
- `--byok-credential-id <credential-uuid>`
- `--notification-config <json>`
- `--language <locale>`
- `--standalone`: do not bind the timer to the current conversation

Notes:

- if `--standalone` is not set and `--source-conversation-id` is omitted, the CLI will try to bind `CURRENT_CONVERSATION_ID` automatically
- `--create-new-task` and `--append-to-source` choose where each run lands: a fresh task per run, or appended to the bound source conversation. Both exist on `create` and `update`.
- `--project-id` must be a knowledge-base/project block id; invalid stored project bindings are ignored at execution time so run history stays on accessible conversations
- when `timer create` binds a source conversation and `--model-override` is omitted, the server inherits concrete model selections but folds stale or invalid snapshots to `auto`
- when the bound source conversation uses BYOK and the flag is omitted, the server inherits that exact credential; pass `--byok-credential-id` to make a different explicit selection
- `--model-override` accepts `auto` or a concrete model id; invalid model selections fail in the CLI, and the server owns the current model allowlist, routing, and plan gates
- BYOK credentials are created through `kollab space byok upsert`; workspace ownership and entitlement are checked before the timer is persisted
- for detailed timer inspection, use `list` and `history`

## MCP connector scope

Timer MCP scope is a three-state decision:

- Omit `--mcp-server-ids` only when the user wants the backend default all-connectors scope. This lets the timer use every currently available connector and future connectors that become available.
- Pass `--mcp-server-ids '[]'` when the timer should run without MCP connectors.
- Pass a comma-separated list or JSON array when the timer should use only specific connectors, for example `--mcp-server-ids mcp-x,mcp-slack` or `--mcp-server-ids '["mcp-x","mcp-slack"]'`.

Before creating a timer, inspect/decide whether the per-run prompt needs external services. Use `kollab mcp connected` or `kollab mcp list` to find available connector ids when needed. If the user's intent does not clearly say all connectors, no connectors, or specific connectors, ask once before running `kollab timer create`; do not silently rely on the default all-connectors behavior.


## Update

```bash
kollab timer update --timer-id <timer-id> --title "New Title" --timezone Asia/Shanghai --schedule-ui-config '{"days":[1,3,5],"time":"11:00"}'
kollab timer update --timer-id <timer-id> --clear-byok
```

`update` supports roughly the same fields as `create`; `--standalone` applies only at `create` time. Use `--clear-byok` to return a timer to platform model routing without deleting the workspace credential.

## Run Control and Delete

```bash
kollab timer pause --timer-id <timer-id>
kollab timer resume --timer-id <timer-id>
kollab timer run-now --timer-id <timer-id>
kollab timer delete --timer-id <timer-id>
```
