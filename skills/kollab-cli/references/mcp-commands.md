# mcp commands

MCP commands search marketplace connectors, inspect sanitized connection status,
and manage workspace MCP server configurations.

## Querying

```bash
kollab mcp search instagram
kollab mcp search --query instagram --limit 5
kollab mcp connected
kollab mcp list
kollab mcp info --id <mcp-id>
```

Use `search` when the agent needs the correct marketplace `registryId` for an app
name. Its output is safe for model context and includes public catalog metadata:
`registryId`, `name`, `description`, `category`, `connectionType`, `enabled`,
`requestable`, and non-secret Composio fields such as `toolkit`.

Use `connected`, `list`, or `info` when the agent needs to know which MCP connectors are already
installed in the current space. Their output is safe for model context and includes
only non-sensitive status fields: id, name, registryId, connectionType, enabled,
status, statusReason, statusCheckedAt, and whether Composio has a connected
account. It does **not** output config URLs, headers, env vars, tokens,
user-authored descriptions, raw error messages, account ids, auth config ids,
MCP server ids, or metadata signatures.

## Create and Update

```bash
kollab mcp create --name "context7" --type sse --config '{"url":"https://example.com/mcp"}' --description "Docs MCP"

kollab mcp update --id <mcp-id> --enabled false
```

Notes:

- `--config` must be a full JSON object and is suitable only for credential-free connectors. Never place tokens, passwords, headers, env secrets, or secret-bearing URLs in this option because process listings and shell history can retain its argv value.
- `update` sends `PUT /skills/mcp/:id` with only the top-level flags you provide. For any connector that already has credentials, do not use `--config` even when changing a non-secret field: `list`/`info` intentionally cannot reconstruct the private config, and the service requires the Web reconnect flow for a credential-bearing config replacement. Top-level fields such as `--enabled`, `--name`, and `--description` remain safe to update independently.
- the current CLI automatically uses the authenticated `space_id`, so `--space-id` is handled for you
- for assistant-message connector cards, emit the registry id from `search`, e.g.:

````text
```kollab-mcp-connect
{"registryId":"composio-instagram"}
```
````

## Delete

```bash
kollab mcp delete --id <mcp-id>
```

## OAuth Providers and Authorization

```bash
# List all configured OAuth providers and their availability
kollab mcp oauth-providers

# Get the authorization URL for a provider (open in browser)
kollab mcp oauth-start --provider <slug>
kollab mcp oauth-start --provider google-drive
```

Notes on the CLI OAuth flow:

- `oauth-providers` calls `GET /api/mcp-oauth/providers` (public, no auth needed); returns `{ providers: [{ slug, name, enabled }] }`
- `oauth-start` calls `GET /api/mcp-oauth/start/:provider?frontend_origin=...`; the server validates the provider slug and returns a redirect to the provider's authorization page
- **The CLI cannot open a browser popup or receive a postMessage callback.** It prints the `authorizationUrl` with instructions for the user to open it manually in a browser. After the user completes OAuth in the browser, the Kollab Connect Apps dialog updates the connector status.
- Use `--provider` with the `slug` from `oauth-providers`; slugs are lowercase alphanumeric + hyphens (e.g. `google-drive`, `notion`, `github`)
- This is the correct "print URL + manual step" pattern for headless/agent use; the Web UI automates the popup flow

### Keep support and failure surfaces separate

- A cloud CLI error from `kollab mcp ...` is not evidence that webpage OAuth
  failed. Report the CLI/server error on its own; provider consent, callback,
  and popup failures belong to the browser flow.
- Webpage OAuth does not require a Kollab CLI or other tool installation on
  Windows. The CLI's printed authorization URL is an optional headless path,
  not a prerequisite for browser sign-in.
- A cancellation record does not prove that a refund was issued. Do not infer
  or invent refund policy; for an unclear account record, direct the user to
  the official Kollab team at `team@kollab.im`.

## Composio API_KEY Toolkit Connect Flow

For toolkits that authenticate with an API key (rather than OAuth), use the three-step flow:

```bash
# Step 1: get the required credential field names
kollab mcp composio-schema --toolkit <slug>
# e.g.: kollab mcp composio-schema --toolkit hubspot

# Step 2: connect with credentials (field names from schema)
kollab mcp composio-connect --toolkit <slug> --credentials '{"<field_name>":"<value>"}'
# e.g.: kollab mcp composio-connect --toolkit hubspot --credentials '{"generic_api_key":"hb-..."}'

# Step 3: create the MCP server row using placeholder_url from step 2
kollab mcp create --name "HubSpot" --type http --config '{"url":"<placeholder_url from step 2>"}'
```

Notes on Composio API_KEY connect:

- `composio-schema` calls `GET /api/skills/mcp/composio/{toolkit}/credential-schema`; returns `{ toolkit, auth_config_id, fields: [{ name, display_name, description, type, required, default }] }`. Use the `name` field values as keys in `--credentials`.
- `composio-connect` calls `POST /api/skills/mcp/composio/{toolkit}/connect-with-key` with body `{ credentials: { "<field>": "<value>", ... } }`; returns `{ toolkit, connected_account_id, status, placeholder_url, auth_config_id, mcp_server_id, metadata_signature }`.
- **Credential masking**: `composio-connect` replaces all submitted credential values with `****last4` in the output for safe logging. The raw values are sent only to the server.
- If the toolkit requires OAuth rather than an API key, this endpoint returns an error; use `mcp oauth-start` instead.
- `--credentials` must be a JSON object where all values are strings; use `composio-schema` to get the exact key names.

## Catalog Access Request

```bash
# Request operator enablement for a disabled marketplace connector
kollab mcp catalog-request --registry-id <registryId>
kollab mcp catalog-request --registry-id composio-hubspot --name "HubSpot" --reason "CRM integration for sales team"
```

Notes:

- calls `POST /api/skills/mcp/catalog/request` with body `{ registry_id: "<id>" }`
- `--name` and `--reason` are informational-only; they are logged to stderr locally but are **not** sent to the server (the server DTO only accepts `registry_id`)
- returns `{ registry_id, status: "requested" | "already_enabled" }`
- use `kollab mcp search <connector-name>` first to find the `registryId`; it is the `registryId` field in search results
- requires a space-bound token; the server enforces a `spaceId` check
