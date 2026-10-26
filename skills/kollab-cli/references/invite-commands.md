# invite commands

The `invite` group provides space membership invite preview and acceptance.
These are token-based cross-space operations — no `--space-id` is required.

## Preview an Invite

```bash
# Check invite validity and see target space / inviter before accepting
kollab invite preview --token <invite-token>
```

This endpoint is **public** (no authentication required). The command resolves the
base API URL from `KOLLAB_API_URL` or stored credentials. Call this first to confirm
the token is valid and not expired before accepting.

Response fields (InvitePreviewResponseDto): `spaceName`, `inviterDisplayName`,
`inviterAvatarUrl`, `expiresAt`, `isExpired`, `role`.

## Accept an Invite

```bash
# Join the target space (authentication required)
kollab invite accept --token <invite-token>
```

Accepts the invite and adds the authenticated user to the target space. Returns
`AcceptInviteResponseDto` containing the joined `spaceId` and membership details.

Accepting an expired or already-used token returns a 400 error from the server.
Always call `invite preview` first to confirm validity.

## Gating

`invite` is subject to `KOLLAB_ALLOWED_CLI_COMMANDS` deny-by-default gating
(same as `project`, `task`, `space`, etc.). If the workspace bot has a restricted
allowed list that does not include `invite`, both `preview` and `accept` are blocked.
