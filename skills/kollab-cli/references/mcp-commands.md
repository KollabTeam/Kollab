# mcp commands

MCP commands list, inspect, create, update, and delete workspace MCP server configurations.

## Querying

```bash
kollab mcp list
kollab mcp info --id <mcp-id>
```

## Create and Update

```bash
kollab mcp create --name "context7" --type sse --config '{"url":"https://example.com/mcp"}' --description "Docs MCP"

kollab mcp update --id <mcp-id> --enabled false --config '{"url":"https://example.com/new"}'
```

Notes:

- `--config` must be a full JSON object
- `update` submits exactly the fields you explicitly pass
- the current CLI automatically uses the authenticated `space_id`, so `--space-id` is handled for you

## Delete

```bash
kollab mcp delete --id <mcp-id>
```
