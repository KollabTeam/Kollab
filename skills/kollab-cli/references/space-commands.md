# space commands

Space commands cover space CRUD, members, invitations, permission groups, usage, billing, and the recycle bin.

## Basic Operations

```bash
kollab space list
kollab space info
kollab space create --title "My Space"
kollab space create --title "My Space" --icon '{"type":"emoji","value":"🚀"}'
kollab space rename --title "New Space Name"
kollab space update --title "Another Name"
kollab space update --icon '{"type":"emoji","value":"🚀"}'
kollab space reorder --prev-id <prev-space-id> --next-id <next-space-id>
kollab space delete --confirm
```

## Usage and Limits

```bash
kollab space usage
kollab space usage-users --limit 20
kollab space usage-user --user-id <user-id>
kollab space usage-conversation --conversation-id <conversation-id>
kollab space usage-logs --limit 50 --offset 0
kollab space usage-logs --user-id <user-id> --start-time <ms> --end-time <ms> --limit 100 --offset 0
kollab space usage-logs --conversation-id <conversation-id>
kollab space usage-daily --days 30
```

`usage` returns information such as:

- credits / subscription credits / top-up credits / free credits
- storage usage and remaining quota
- seats / concurrent sessions / scheduled tasks
- current billing period

`usage-conversation` returns token and credit usage for a single conversation. All other usage commands operate at the space level.

`usage-logs` returns a paginated log of individual usage events. All filters are optional and combinable:
- `--user-id` — filter by a specific member
- `--conversation-id` — filter by a specific conversation
- `--start-time` / `--end-time` — time range in milliseconds since epoch
- `--limit` — page size (default 50)
- `--offset` — pagination offset (default 0)

## Members and Invitations

```bash
kollab space members --search alice --role admin
kollab space members --department-id <dept-uuid> --status active --page 1 --page-size 50
kollab space member --id <member-uuid>
kollab space member-me
kollab space member-stats
kollab space update-member --id <member-uuid> --role admin
kollab space update-member --id <member-uuid> --name "Alice" --title "Staff Engineer" --department-id <dept-uuid>
kollab space remove-member --id <member-uuid>
kollab space leave
kollab space transfer-ownership --target-member-uuid <member-uuid>
kollab space invite
```

Notes:

- `space members` filters are all optional and combinable: `--search`, `--department-id`, `--status`, `--role`, `--page`, `--page-size`
- role values are lowercase: `owner` | `admin` | `member` | `readonly`; member status values are `pending` | `active`. The server rejects other casing with a 400.
- `space invite` generates an invitation link
- the current CLI follows an invite-link model: add members by sharing the generated invite link
- most space commands default to the authenticated current space; `space list/create` work without a current `space_id`

## Permission Groups

```bash
kollab space permission-groups
kollab space create-permission-group --name "Editors" --user-ids "user-a,user-b"
kollab space update-permission-group --group-id <group-id> --user-ids '["user-a","user-c"]'
kollab space delete-permission-group --group-id <group-id>
```

## Departments

```bash
# List all departments (flat list)
kollab space departments

# List departments as a hierarchical tree
kollab space departments --tree

# Create a top-level department
kollab space create-department --name "Engineering"

# Create a nested department under a parent
kollab space create-department --name "Backend" --parent-id <parent-dept-uuid>

# Assign a department leader at creation time
kollab space create-department --name "Design" --leader-id <member-uuid>

# Update department name, parent, or leader
kollab space update-department --id <dept-uuid> --name "Frontend"
kollab space update-department --id <dept-uuid> --parent-id <new-parent-uuid>
kollab space update-department --id <dept-uuid> --leader-id ""  # clear leader

# Delete a department (irreversible — requires --confirm)
kollab space delete-department --id <dept-uuid> --confirm
```

Notes:

- `departments` returns `DepartmentResponseDto[]` with fields: `uuid`, `name`, `parentId`, `leaderId`, `memberCount`, `createdAt`, `updatedAt`.
- `departments --tree` returns the same fields plus a `children` array at each node.
- `create-department` and `update-department` accept `--leader-id ""` (empty string) to explicitly clear the leader assignment; omitting `--leader-id` leaves it unchanged.
- `delete-department` does NOT cascade to child departments — detach or delete children first.
- All department commands operate on the current space from credentials.

## Billing and Top-ups

```bash
kollab space billing-subscription
kollab space billing-catalog
kollab space billing-checkout --plan pro
kollab space billing-portal
kollab space billing-cancel-subscription
kollab space billing-topup --package-id topup_3000
```

Notes:

- plan upgrades only support `pro` and `max`
- top-up packages currently include `topup_1500`, `topup_3000`, and `topup_15000`
- test environments may additionally use `--test-clock-id`

## BYOK Models

```bash
kollab space byok list
kollab space byok get --id <credential-uuid>
kollab space byok upsert \
  --model-id claude-sonnet-4-6 --base-url https://api.anthropic.com \
  --api-key-env KOLLAB_BYOK_API_KEY
kollab space byok upsert \
  --provider openai-compatible --api-format chat-completions \
  --model-id gpt-5.2 --base-url https://api.openai.com \
  --api-key-env OPENAI_BYOK_API_KEY
kollab space byok upsert \
  --provider openai-compatible --api-format responses \
  --model-id gpt-5.2 --base-url https://api.openai.com \
  --api-key-env OPENAI_BYOK_API_KEY
kollab space byok rotate --id <credential-uuid> \
  --model-id claude-sonnet-4-6 --base-url https://api.anthropic.com \
  --api-key-file /secure/path/key.txt
kollab space byok enable --id <credential-uuid>
kollab space byok disable --id <credential-uuid>
kollab space byok test --id <credential-uuid>
kollab space byok delete --id <credential-uuid> --confirm
```

Notes:

- Responses are masked JSON and omit plaintext, ciphertext, fingerprints, and KMS ids.
- Kollab Agent BYOK accepts Anthropic Messages, OpenAI Chat Completions, and OpenAI Responses endpoints. The server translates both OpenAI formats back to the Anthropic stream consumed by Claude Agent SDK.
- `--provider openai-compatible` requires the provider's actual wire format through `--api-format chat-completions|responses` when creating a credential. Updating or rotating by `--id` preserves the stored provider/format when those flags are omitted.
- Secrets come only from a named environment variable or UTF-8 file; env wins when both are supplied.
- `upsert` / `rotate` accept `--disabled` to save the credential in a disabled state; toggle later with `byok enable` / `byok disable`.
- `test` fails closed until the secure gateway executor is configured and never returns a fake pass.
- Pro/Max use provider-neutral multi-credential management. AppSumo/admin sources use the separate
  single-credential lane only when compatibility is enabled; dormant rows remain masked.

## Billing Invoices

```bash
kollab space billing-invoices
```

Notes:

- fetches Stripe invoice history for the current space
- no pagination parameters — all invoices are returned in a single response
- requires admin or owner membership in the space

## Membership Redeem

```bash
kollab space redeem --code <code>
```

Notes:

- redeems a membership code (6–32 characters) and applies it to the current space
- returns `{ spaceId, planType, code, currentPeriodStart, currentPeriodEnd }` on success
- a code can only be redeemed once globally; subsequent attempts return an error
- if you need to target a different space, set `KOLLAB_SPACE_ID` before running this command

## Referral

```bash
# Create a referral invite link for the current space
kollab space referral-create

# Get referral statistics (invited count, credits earned)
kollab space referral-stats
```

Notes:

- `referral-create` calls `POST /spaces/:spaceId/referrals/invites` and returns
  `{ token, inviteLink, inviteExpiresAt?, emailSent? }`. The `inviteLink` is the
  shareable URL; share it manually — email delivery is not supported via CLI.
- `referral-stats` calls `GET /spaces/:spaceId/referrals/stats` and returns
  `{ referralCount, creditsEarned, rewardLimit, rewardRemaining }`.
- Rewards are applied when an invited user completes registration via the invite link.

## Recycle Bin (Trash)

```bash
# List deleted blocks (paginated)
kollab space trash-list
kollab space trash-list --page 2 --limit 50

# Restore a deleted block to its original location
kollab space trash-restore --block-id <block-uuid>

# Permanently delete one block (irreversible — requires --confirm)
kollab space trash-delete --block-id <block-uuid> --confirm

# Empty the entire recycle bin (irreversible — requires --confirm)
kollab space trash-empty --confirm
```

Notes:

- `trash-list` returns `{ items, total, page, limit, hasMore }`. Each item has `uuid`, `title`, `size`, `deletedAt` (ms timestamp), `deletedBy`, and `parentTitle`.
- `trash-restore` returns a deleted block to its original parent location. Returns 404 if the block is not in the trash.
- `trash-delete` and `trash-empty` are irreversible; both require `--confirm` to prevent accidental data loss, consistent with `space delete` and `project delete`.
- `trash-empty` returns `{ message, deletedCount }`.

## Deleted Space Recovery

List and restore soft-deleted spaces. Unlike the Recycle Bin (block-level, within a space), these commands operate at the **space level** and are scoped to the current user (cross-space, no `--space-id` required for listing).

Spaces are retained for 30 days after deletion. Only the original Owner can restore.

```bash
# List all soft-deleted spaces owned by the current user (30-day retention)
kollab space deleted-list

# Restore a soft-deleted space (also cascade-restores knowledge bases deleted at the same time)
kollab space restore --uuid <space-uuid>
```

Notes:

- `deleted-list` returns `SpaceDto[]` ordered by `deletedAt` DESC. Each item has `uuid`, `title`, `icon`, `deletedAt` (ms timestamp). Empty array if no deleted spaces.
- `restore` returns the restored `SpaceDto` with `status` back to `NORMAL` and `deletedAt` cleared to `null`. Returns 403 if not the owner, 404 if not found or not archived, 410 if past the 30-day retention window.
