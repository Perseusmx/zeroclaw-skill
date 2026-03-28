# ZeroClaw Channels Reference

This reference was refreshed against the official ZeroClaw channels docs on **March 28, 2026**.

## Quick Reference

```bash
zeroclaw channel list
zeroclaw channel start
zeroclaw channel doctor
```

Channel startup is best-effort: one channel can fail initialization while others still start.

## Security Model

All channels are deny-by-default:
- Empty allowlist: deny all inbound messages
- `"*"`: allow all inbound senders temporarily
- Explicit list: allow only the listed senders

Use `"*"` only for verification, then tighten it immediately.

## Runtime Chat Commands

Telegram and Discord support per-sender runtime routing:
- `/models`
- `/models <provider>`
- `/model`
- `/model <model-id>`
- `/new`

Non-CLI channels also support supervised approval commands:
- `/approve-request <tool>`
- `/approve-confirm <request-id>`
- `/approve-allow <tool>`
- `/approve-deny <request-id>`
- `/approve-pending`
- `/approve <tool>`
- `/unapprove <tool>`
- `/approvals`

Natural-language approval intents can also be enabled through `[autonomy].non_cli_natural_language_approval_mode`.

## Channel Matrix

| Channel | Receive Mode | Public Port Required | Allowlist Field |
|---|---|---|---|
| CLI | local stdin/stdout | No | n/a |
| Telegram | polling | No | `allowed_users` |
| Discord | gateway/websocket | No | `allowed_users` |
| Slack | Events API | No | `allowed_users` |
| Mattermost | polling | No | `allowed_users` |
| Matrix | sync API | No | `allowed_users` |
| Signal | signal-cli HTTP bridge | No | `allowed_from` |
| WhatsApp | webhook or websocket | Cloud API only | `allowed_numbers` |
| WATI | webhook | Yes | `allowed_numbers` |
| Webhook | gateway endpoint | Usually yes | `secret` |
| Email | IMAP polling + SMTP | No | `allowed_senders` |
| IRC | IRC socket | No | `allowed_users` |
| Lark | websocket or webhook | Webhook mode only | `allowed_users` |
| Feishu | websocket or webhook | Webhook mode only | `allowed_users` |
| DingTalk | stream mode | No | `allowed_users` |
| QQ | bot gateway | No | `allowed_users` |
| Napcat | websocket + HTTP send | No | `allowed_users` |
| Linq | webhook | Yes | `allowed_senders` |
| iMessage | local integration | No | `allowed_contacts` |
| Nextcloud Talk | webhook | Yes | `allowed_users` |
| ACP | stdio JSON-RPC 2.0 | No | `allowed_users` |
| Nostr | relay websocket | No | `allowed_pubkeys` |

## Common Config Shapes

### Telegram

```toml
[channels_config.telegram]
bot_token = "123456:telegram-token"
allowed_users = ["*"]
stream_mode = "off" # off | partial | on
draft_update_interval_ms = 1000
mention_only = false
interrupt_on_new_message = false
ack_enabled = true

[channels_config.telegram.group_reply]
mode = "all_messages" # or "mention_only"
allowed_sender_ids = []
```

### Discord

```toml
[channels_config.discord]
bot_token = "discord-bot-token"
guild_id = "123456789012345678" # optional
allowed_users = ["*"]
listen_to_bots = false
mention_only = false

[channels_config.discord.group_reply]
mode = "all_messages"
allowed_sender_ids = []
```

### Slack

```toml
[channels_config.slack]
bot_token = "xoxb-..."
app_token = "xapp-..." # optional
channel_id = "*"       # omit or "*" for all accessible channels
allowed_users = ["*"]

[channels_config.slack.group_reply]
mode = "all_messages"
allowed_sender_ids = []
```

### Matrix

```toml
[channels_config.matrix]
homeserver = "https://matrix.example.com"
access_token = "syt_..."
user_id = "@zeroclaw:matrix.example.com" # recommended for E2EE
device_id = "DEVICEID123"                # recommended for E2EE
room_id = "!room:matrix.example.com"
allowed_users = ["*"]
mention_only = false
```

Matrix support may require a build with `channel-matrix`.

### WhatsApp

Cloud API:

```toml
[channels_config.whatsapp]
access_token = "EAAB..."
phone_number_id = "123456789012345"
verify_token = "your-verify-token"
app_secret = "your-app-secret"
allowed_numbers = ["*"]
```

WhatsApp Web:

```toml
[channels_config.whatsapp]
session_path = "~/.zeroclaw/state/whatsapp-web/session.db"
pair_phone = "15551234567" # optional
pair_code = ""             # optional
allowed_numbers = ["*"]
```

### Webhook

```toml
[channels_config.webhook]
port = 8080
secret = "optional-shared-secret"
```

### Email

```toml
[channels_config.email]
imap_host = "imap.example.com"
imap_port = 993
imap_folder = "INBOX"
smtp_host = "smtp.example.com"
smtp_port = 465
smtp_tls = true
username = "bot@example.com"
password = "email-password"
from_address = "bot@example.com"
poll_interval_secs = 60
allowed_senders = ["*"]
imap_id = { enabled = true, name = "zeroclaw", version = "0.6.5", vendor = "zeroclaw-labs" }
```

### Lark

```toml
[channels_config.lark]
app_id = "your_lark_app_id"
app_secret = "your_lark_app_secret"
encrypt_key = ""
verification_token = ""
allowed_users = ["*"]
receive_mode = "websocket" # or "webhook"
port = 8081                # required for webhook mode

[channels_config.lark.group_reply]
mode = "all_messages"
allowed_sender_ids = []
```

### Feishu

```toml
[channels_config.feishu]
app_id = "your_feishu_app_id"
app_secret = "your_feishu_app_secret"
encrypt_key = ""
verification_token = ""
allowed_users = ["*"]
receive_mode = "websocket" # or "webhook"
port = 8081

[channels_config.feishu.group_reply]
mode = "all_messages"
allowed_sender_ids = []
```

### QQ

```toml
[channels_config.qq]
app_id = "qq-app-id"
app_secret = "qq-app-secret"
allowed_users = ["*"]
receive_mode = "webhook" # default; websocket is legacy fallback
environment = "production"
```

### Napcat

```toml
[channels_config.napcat]
websocket_url = "ws://127.0.0.1:3001"
api_base_url = "http://127.0.0.1:3001"
access_token = ""
allowed_users = ["*"]
```

### Nextcloud Talk

```toml
[channels_config.nextcloud_talk]
base_url = "https://cloud.example.com"
app_token = "nextcloud-talk-app-token"
webhook_secret = "optional-webhook-secret"
allowed_users = ["*"]
```

### Linq

```toml
[channels_config.linq]
api_token = "linq-partner-api-token"
from_phone = "+15551234567"
signing_secret = "optional-webhook-signing-secret"
allowed_senders = ["*"]
```

### WATI

```toml
[channels_config.wati]
api_token = "wati-api-token"
api_url = "https://live-mt-server.wati.io"
webhook_secret = "required-shared-secret"
tenant_id = "tenant-id"
allowed_numbers = ["*"]
```

### ACP

```toml
[channels_config.acp]
opencode_path = "opencode"
workdir = "/path/to/workspace"
extra_args = []
allowed_users = ["*"]
```

## Validation Workflow

1. Configure one channel with `"*"` temporarily.
2. Run `zeroclaw onboard --channels-only`.
3. Run `zeroclaw daemon`.
4. Send a test message from the expected sender.
5. Replace `"*"` with explicit IDs or contacts.

## Troubleshooting

If a channel appears connected but does not respond:
1. Check the correct allowlist field.
2. Check bot membership and permissions.
3. Check tokens, secrets, and callback URLs.
4. For webhook channels, confirm reachable HTTPS.
5. Restart `zeroclaw daemon` after config changes.

For fast log capture:

```bash
RUST_LOG=info zeroclaw daemon 2>&1 | tee /tmp/zeroclaw.log
rg -n "Telegram|Discord|Slack|Mattermost|Matrix|Signal|WhatsApp|Email|IRC|Lark|Feishu|DingTalk|QQ|Napcat|Nostr|ACP|Webhook|Channel" /tmp/zeroclaw.log
```
