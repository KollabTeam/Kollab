# space commands

Space commands cover space CRUD, members, invitations, permission groups, usage, and billing.

## Basic Operations

```bash
kollab space list
kollab space info
kollab space create --title "My Space"
kollab space rename --title "New Space Name"
kollab space update --title "Another Name"
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
kollab space usage-daily --days 30
```

`usage` returns information such as:

- credits / subscription credits / top-up credits / free credits
- storage usage and remaining quota
- seats / concurrent sessions / scheduled tasks
- current billing period

## Members and Invitations

```bash
kollab space members --search alice --role ADMIN
kollab space member --id <member-uuid>
kollab space member-me
kollab space member-stats
kollab space update-member --id <member-uuid> --role ADMIN
kollab space remove-member --id <member-uuid>
kollab space leave
kollab space transfer-ownership --target-member-uuid <member-uuid>
kollab space invite
```

Notes:

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
