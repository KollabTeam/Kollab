# timer commands

Timer commands create, inspect, update, and delete scheduled runs.

## Querying

```bash
kollab timer list
kollab timer history --timer-id <timer-id> --page 1 --page-size 20
```

## Create

```bash
kollab timer create --title "Daily Check" --prompt "Inspect recent errors in the test environment" --schedule-kind cron --cron '0 10 * * 1-5' --timezone Asia/Shanghai --schedule-ui-mode builder --schedule-ui-config '{"days":[1,2,3,4,5],"time":"10:00"}'
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
- `--mcp-server-ids <csv-or-json-array>`
- `--knowledge-base-scope <csv-or-json-array>`
- `--agent-id <id>`
- `--model-override <model>`
- `--notification-config <json>`
- `--language <locale>`
- `--standalone`: do not bind the timer to the current conversation

Notes:

- if `--standalone` is not set and `--source-conversation-id` is omitted, the CLI will try to bind `CURRENT_CONVERSATION_ID` automatically
- `--project-id` must be a knowledge-base/project block id; invalid stored project bindings are ignored at execution time so run history stays on accessible conversations
- when `timer create` binds a source conversation and `--model-override` is omitted, the server will inherit that conversation's `modelTier`
- for detailed timer inspection, use `list` and `history`

## Update

```bash
kollab timer update --timer-id <timer-id> --title "New Title" --timezone Asia/Shanghai --schedule-ui-config '{"days":[1,3,5],"time":"11:00"}'
```

`update` supports roughly the same fields as `create`; `--standalone` applies only at `create` time.

## Run Control and Delete

```bash
kollab timer pause --timer-id <timer-id>
kollab timer resume --timer-id <timer-id>
kollab timer run-now --timer-id <timer-id>
kollab timer delete --timer-id <timer-id>
```
