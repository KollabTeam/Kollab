# bot commands

Bot commands cover platform configuration, knowledge scope, CLI permissions, model tier, enable/disable operations, and conversation / message lookup.

## Querying and Details

```bash
kollab bot list --platform feishu --enabled true
kollab bot info --id <bot-id>
kollab bot credentials --id <bot-id>
kollab bot conversations --id <bot-id> --page 1 --page-size 20
kollab bot messages --id <bot-id> --conversation-id <conversation-id>
```

## Create and Update

```bash
kollab bot create \
  --name "Ops Bot" \
  --platform feishu \
  --kb-ids "kb-a,kb-b" \
  --cli-permissions all \
  --model-tier pro

kollab bot update \
  --id <bot-id> \
  --platform-config '{"feishu":{"enabled":true}}' \
  --cli-permissions task:read,artifact:read \
  --model-tier default
```

Key parameters:

- `--kb-ids <csv-or-json-array>`
- `--avatar <json>`
- `--platform-config <json>`
- `--cli-permissions <all|none|csv>`
- `--model-tier <lite|pro|max|default>`

Notes:

- `default` clears the bot-specific model tier and falls back to the space plan default
- free spaces should assume only `lite` is reliably available; higher tiers may be blocked server-side
- `cli-permissions all` maps to `allowedCliCommands = null`
- `cli-permissions none` maps to `allowedCliCommands = []`
- Provider-backed social data spends paid provider quota; access it through the AgentCore
  `/kollab-social` skill, which is the sanctioned channel for it.

## Enable / Disable / Delete

```bash
kollab bot toggle --id <bot-id> --enabled true
kollab bot delete --id <bot-id>
```
