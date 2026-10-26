# bot commands

Bot commands cover platform configuration, knowledge scope, CLI permissions, model selection, enable/disable operations, conversation / message lookup, and WeChat ClawBot QR login binding.

Supported platforms: `feishu`, `slack`, `telegram`, `wecom`, `weixin` (WeChat ClawBot).

## Contents

- [Querying and Details](#querying-and-details) — list, info, credentials, conversations, messages
- [Create and Update](#create-and-update) — creating bots and editing config
- [Enable / Disable / Delete](#enable--disable--delete) — lifecycle operations
- [Feishu Bot (platform: feishu) — scan-to-create](#feishu-bot-platform-feishu--scan-to-create) — self-service provisioning
- [WeChat ClawBot (platform: weixin)](#wechat-clawbot-platform-weixin) — QR login binding

## Querying and Details

```bash
kollab bot list --platform feishu --enabled true
kollab bot list --search "Ops" --page 1 --page-size 20
kollab bot list --platform weixin
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
  --model-tier auto \
  --byok-credential-id <credential-uuid>

# WeChat ClawBot: no appId/appSecret required at create time
# Credentials arrive later via QR login (see weixin-login below)
kollab bot create \
  --name "WeChat Bot" \
  --platform weixin \
  --model-tier auto

kollab bot update \
  --id <bot-id> \
  --platform-config '{"feishu":{"enabled":true}}' \
  --cli-permissions task:read,artifact:read \
  --model-tier auto

kollab bot update --id <bot-id> --clear-byok
```

Key parameters:

- `--app-id <appId>` / `--app-secret <secret>`: platform app credentials (not needed for `weixin`, which binds via QR login)
- `--skill-ids <csv>`: skills the bot may use
- `--mcp-ids <csv>`: MCP connectors the bot may use
- `--kb-ids <csv-or-json-array>`
- `--prompt <text>`: custom system prompt defining the bot role
- `--avatar <json>`
- `--platform-config <json>`
- `--cli-permissions <all|none|csv>`
- `--model-tier <auto|model-id>`
- `--byok-credential-id <credential-uuid>`
- `--clear-byok` (update only)

Notes:

- Use explicit `--model-tier auto` for ordinary Bot acceptance.
- concrete model ids are validated server-side, including paid-model plan gates; invalid model selections fail in the CLI before a request is sent
- BYOK uses a credential created by `kollab space byok upsert`; the server validates workspace ownership and entitlement before persisting it
- `--clear-byok` removes the Bot credential selection without deleting the workspace credential
- `cli-permissions all` maps to `allowedCliCommands = null`
- `cli-permissions none` maps to `allowedCliCommands = []`
- Provider-backed social data spends paid provider quota; use `kollab social fetch` with an account API key.


## Enable / Disable / Delete

```bash
kollab bot toggle --id <bot-id> --enabled true
kollab bot delete --id <bot-id>
```

---

## Feishu Bot (platform: feishu) — scan-to-create

### Overview

A Feishu Bot is created by scanning one QR code. Kollab creates the Feishu application, requests its
permissions, subscribes the message event, and publishes the version server-side. The user never opens the
Feishu developer console and never sees or supplies an App ID, App Secret, verification token, or encrypt key.

Use this for every new Feishu Bot. Manual Feishu creation is no longer offered in the product, so
`kollab bot create --platform feishu` is for pre-existing manually configured apps only — it does not
produce a working Bot on its own.

Only a workspace administrator can run this flow, and only the account that started a registration can
read, retry, or cancel it.

### Conversation-driven creation flow

```
1. Create the registration → obtain the QR link
2. Give the link to the user (as an image or a clickable link) and ask them to scan it in Feishu
3. Poll until ready
4. Tell the user the Bot is live and they can message it in Feishu
```

#### Step 1 — Create the registration and present the QR to the user

```bash
# Agent-runtime recommended: return as soon as the QR exists, do not hold the turn open
kollab bot feishu-create --name "Ops Bot" --no-wait
# stdout JSON: { uuid: "<registration-id>", status: "waiting_for_scan", qrCodeUrl: "https://open.feishu.cn/..." }
```

Present `qrCodeUrl` to the user as a QR image or a link they can tap. Do not ask them to copy text out of a
terminal. The link expires in roughly 10 minutes; if it lapses, create a new registration.

#### Step 2 — Poll until the setup finishes

```bash
kollab bot feishu-status <registration-id>          # one read
kollab bot feishu-status <registration-id> --wait   # poll to a terminal state
```

Blocking variant, when holding the turn is acceptable:

```bash
kollab bot feishu-create --name "Ops Bot"   # creates, prints the QR, then polls to a terminal state
```

#### Step 3 — Resume, repair, or abandon

```bash
kollab bot feishu-list                       # unfinished registrations owned by this user
kollab bot feishu-cancel <registration-id>   # abandon and scrub the stored Feishu credentials
kollab bot feishu-resync <bot-id>            # re-run provisioning on the SAME Feishu application
```

`feishu-resync` takes the Bot UUID and covers two cases: retrying a failed setup, and repairing a Bot that
reports success but receives nothing. Feishu can accept a configuration write without making it live, so a
ready Bot is not proof that messages will arrive. Prefer resync over deleting the Bot: deleting and
re-scanning leaves an orphaned Feishu application behind. The Bot is disabled while the resync runs and is
re-enabled on success; a failed resync leaves it disabled.

### Statuses

| Status | Meaning | Next action |
| --- | --- | --- |
| `waiting_for_scan` | QR issued, waiting for the user | Keep polling; re-create if the QR expires |
| `configuring` | Scan done, Kollab is configuring Feishu | Keep polling |
| `awaiting_admin_approval` | The Feishu tenant administrator must approve | Stop polling; after approval run `feishu-status --wait` |
| `ready` | Bot created, published, and enabled | Tell the user to message it in Feishu |
| `failed` | Configuration did not complete | Retry via `feishu-resync`; if it repeats, cancel and re-create |
| `expired` | QR lapsed before it was scanned | Create a new registration |
| `cancelled` | Abandoned by the owner | Create a new registration |

`failureCode: feishu_self_service_qr_regeneration_required` means the existing Feishu application cannot be
repaired at all. Cancel the registration and create a new one; retrying it can never succeed.

### Full example (agent script)

```bash
REG=$(kollab bot feishu-create --name "Ops Bot" --no-wait | jq -r '.data.uuid')
QR=$(kollab bot feishu-status "$REG" | jq -r '.data.qrCodeUrl')
# → show $QR to the user as a QR image / tappable link, ask them to scan it in Feishu
kollab bot feishu-status "$REG" --wait | jq '.data.status, .data.botUuid'
```

## WeChat ClawBot (platform: weixin)

### Overview

WeChat ClawBot binds a Kollab Bot to a WeChat account using Tencent's iLink Bot API.
After binding, any WeChat user who DMs the bound account triggers a Kollab agent conversation.
Each WeChat sender maps to their own Kollab conversation (`from_user_id` as the per-user key).

**v1 limits (document these to the user):**

- DM-only: no WeChat group support in v1.
- Inbound: text messages and voice (server-side transcription provided by WeChat/iLink).
- Outbound: text-only (no media, no file attachments in v1).
- Multiple WeChat users can DM one bot concurrently; each gets their own Kollab conversation.
- Token invalidation (e.g. after device change or Tencent revocation) requires a new QR scan.

### Conversation-driven creation flow

This is the recommended agent-driven flow for binding a WeChat ClawBot:

```
1. Create the bot (no credentials needed at this stage)
2. Start login session → obtain QR URL
3. Present the QR URL to the user (as image/link in chat)
4. Poll until confirmed (or handle need_verifycode)
5. Confirm binding to user
```

#### Step 1 — Create bot

```bash
kollab bot create --name "WeChat Bot" --platform weixin --model-tier auto
# Returns: { uuid: "<bot-id>", name: "WeChat Bot", platform: "weixin", ... }
BOT_ID="<bot-id from above>"
```

#### Step 2 — Start QR login and present to user

```bash
# Default: start login + poll until confirmed (blocks up to 8 minutes)
kollab bot weixin-login $BOT_ID

# Agent-runtime recommended: start login without waiting, capture qrcodeUrl
RESULT=$(kollab bot weixin-login $BOT_ID --no-wait)
QR_URL=$(echo "$RESULT" | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['qrcodeUrl'])")
```

After getting `qrcodeUrl`, the agent must present it to the user:

- Display the URL as a clickable link, or
- Render it as an image/QR code in chat so the user can scan with WeChat.

Tell the user: "请用微信扫描此二维码完成绑定" (Please scan this QR code with WeChat to complete binding).

#### Step 3 — Poll login status

```bash
# If using --no-wait above, poll status separately:
kollab bot weixin-login $BOT_ID
# Exits 0 on confirmed, non-zero on error/timeout
```

#### Step 4 — Handle need_verifycode

If WeChat shows a pairing number on the user's phone:

```bash
# User reports the number shown in WeChat (e.g. "123456")
kollab bot weixin-login $BOT_ID --verify-code 123456
```

Tell the user: "请在您的微信中找到配对数字，并将其提供给我" (Please find the pairing number in your WeChat and provide it to me).

### Status codes

| Status                | Meaning                     | Next action                          |
| --------------------- | --------------------------- | ------------------------------------ |
| `wait`                | QR not yet scanned          | Keep polling                         |
| `scaned`              | Phone scanned; verifying    | Keep polling                         |
| `need_verifycode`     | WeChat shows pairing number | Re-run with `--verify-code <number>` |
| `confirmed`           | Bound successfully          | Done                                 |
| `expired`             | QR TTL exceeded             | Re-run `weixin-login` to get new QR  |
| `verify_code_blocked` | Too many wrong codes        | Re-run `weixin-login` to reset       |
| `token_invalid`       | Tencent revoked the token   | Re-run `weixin-login` to re-bind     |

### Full example (agent script)

```bash
# Create
OUTPUT=$(kollab bot create --name "WeChat Bot" --platform weixin --model-tier auto)
BOT_ID=$(echo "$OUTPUT" | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['uuid'])")

# Start login
LOGIN=$(kollab bot weixin-login "$BOT_ID" --no-wait)
QR_URL=$(echo "$LOGIN" | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['qrcodeUrl'])")

# Present QR URL to user
echo "请用微信扫描此链接对应的二维码: $QR_URL"

# Poll to completion (blocks up to 8 min, exits 0 on confirmed)
kollab bot weixin-login "$BOT_ID"
echo "微信 ClawBot 绑定成功！"
```
