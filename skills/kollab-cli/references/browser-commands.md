# browser commands

Browser commands manage the current user's connected browser extension instances.
Browser extensions let Kollab agents control a local browser for web automation tasks.

## List Extensions

```bash
kollab browser list
```

Response fields per extension: `clientId`, `name`, `displayName`, `platform`,
`browserInfo`, `extensionVersion`, `online`, `lastHeartbeat`.

Notes:

- Only extensions that are currently online (heartbeating) appear in the list.
- `displayName` is the user-editable label; `name` is the immutable system alias.
- Use `clientId` for rename and disconnect operations.

## Rename an Extension

```bash
kollab browser rename --id <clientId> --name "Office Chrome"
```

Notes:

- Sets the `displayName` shown in the Connectors page.
- Returns `{ success: true }` on success, 404 if the extension is not found or offline.

## Disconnect an Extension

```bash
# Disconnect an extension (requires --confirm)
kollab browser disconnect --id <clientId> --confirm
```

Notes:

- Notifies the extension via WebSocket to stop its local transport.
- Removes the extension from the live state service and deletes its synthetic MCP records.
- After disconnect, the extension must reconnect manually (reload extension or reopen browser).
- Returns `{ success: true }`.

## Screenshot URL

```bash
# Get a fresh signed download URL for a stored screenshot
kollab browser screenshot-url --object-name temp/screenshot-abc12345.png
```

Notes:

- The `objectName` is the stable storage path returned when a screenshot was originally
  uploaded (stored in browser step history within conversation messages).
- Signed URLs expire, but the objectName is permanent; this command re-signs for fresh access.
- Returns `{ downloadUrl: string }`.
