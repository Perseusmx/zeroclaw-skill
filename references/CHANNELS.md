# ZeroClaw Channels Reference

ZeroClaw supports 16+ communication channels for autonomous AI interaction. This document provides detailed setup instructions for each channel.

**Quick reference:**
```bash
# List all configured channels
zeroclaw channel list

# Start all channels
zeroclaw channel start

# Start specific channel
zeroclaw channel start --channel <channel-name>

# Channel health check
zeroclaw channel doctor
```

## Security Model

All channels use **deny-by-default** access control:
- Empty allowlist: deny all inbound messages
- `"*"`: allow all senders (use for verification only)
- Explicit list: allow only listed senders

<security-warning>
**⚠️ SECURITY**: Always start with `"*"` allowlist for verification, then tighten to explicit IDs. Never keep `"*"` in production configurations.
</security-warning>

## In-Chat Commands

**Telegram and Discord** support runtime model switching:
- `/models` - Show available providers and current selection
- `/models <provider>` - Switch provider for current session
- `/model` - Show current model
- `/model <model-id>` - Switch model for current session
- `/new` - Clear conversation history and start fresh

## Channel Matrix

| Channel | Receive Mode | Public Port Required | Allowlist Field | Quick Setup |
|---|---|---|---|---|
| CLI | Local stdin/stdout | No | N/A | Built-in |
| Telegram | Polling | No | `allowed_users` | `zeroclaw onboard` |
| Discord | Gateway/WebSocket | No | `allowed_users` | `zeroclaw onboard` |
| Slack | Events API | No | `allowed_users` | `zeroclaw onboard` |
| Mattermost | Polling | No | `allowed_users` | Manual config |
| Matrix | Sync API (E2EE) | No | `allowed_users` | Manual config |
| Signal | Signal-cli HTTP | No | `allowed_from` | Manual config |
| WhatsApp | Webhook/WebSocket | Cloud API: Yes, Web: No | `allowed_numbers` | `zeroclaw onboard` / Manual |
| Webhook | Gateway endpoint | Usually yes | N/A (uses `secret`) | Manual/onboard |
| Email | IMAP polling + SMTP | No | `allowed_senders` | Manual config |
| IRC | IRC socket | No | `allowed_users` | `zeroclaw onboard` |
| Lark | WebSocket/Webhook | Webhook mode: Yes | `allowed_users` | Manual config |
| DingTalk | Stream mode | No | `allowed_users` | `zeroclaw onboard` |
| QQ | Bot gateway | No | `allowed_users` | Manual config |
| Linq | Webhook | Yes | `allowed_senders` | Manual config |
| iMessage | Local integration | No | `allowed_contacts` | Manual config |
| Nostr | Relay WebSocket | No | `allowed_pubkeys` | Manual config |

---

## Per-Channel Setup

### Telegram

**Quick setup:**
```bash
zeroclaw onboard
# Follow prompts for Telegram bot token
```

**Manual config:**
```toml
[channels_config.telegram]
bot_token = "123456:your-telegram-bot-token"
allowed_users = ["your-username"]  # or user ID
stream_mode = "off"                # off | partial
draft_update_interval_ms = 1000      # Edit throttle for streaming
mention_only = false                # Require @mention in groups
interrupt_on_new_message = false      # Cancel on new message
```

**Get bot token:**
1. Message @BotFather on Telegram
2. Create new bot
3. Copy the token

**Get user ID:**
1. Message `@userinfobot` on Telegram
2. Copy your numeric user ID

### Discord

**Quick setup:**
```bash
zeroclaw onboard
# Follow prompts for Discord bot token
```

**Manual config:**
```toml
[channels_config.discord]
bot_token = "your-discord-bot-token"
guild_id = "123456789012345678"   # Optional: specific server
allowed_users = ["*"]                   # User IDs
listen_to_bots = false
mention_only = false
```

**Get bot token:**
1. Go to Discord Developer Portal
2. Create application → Bot → Add Bot
3. Enable Message Content Intent
4. Copy the token

**Get user ID:**
1. Enable Developer Mode in Discord
2. Right-click user → Copy User ID

### Slack

**Quick setup:**
```bash
zeroclaw onboard
# Follow prompts for Slack bot token
```

**Manual config:**
```toml
[channels_config.slack]
bot_token = "xoxb-your-bot-token"
app_token = "xapp-your-app-token"       # Optional: Socket Mode
channel_id = "C1234567890"            # Optional: specific channel
allowed_users = ["*"]                   # Member IDs
```

**Listen behavior:**
- `channel_id = "C123..."`: Listen only on that channel
- `channel_id = "*"` or omitted: Listen across all accessible channels

### WhatsApp

ZeroClaw supports two backends:

#### Cloud API Mode

**Config:**
```toml
[channels_config.whatsapp]
access_token = "your-access-token"
phone_number_id = "123456789012345"
verify_token = "your-verify-token"
app_secret = "your-app-secret"          # Recommended
allowed_numbers = ["*"]
```

#### WhatsApp Web Mode

**Config:**
```toml
[channels_config.whatsapp]
session_path = "~/.zeroclaw/state/whatsapp-web/session.db"
pair_phone = "+15551234567"              # Optional
pair_code = ""                          # Optional
allowed_numbers = ["*"]
```

### Matrix

**Config:**
```toml
[channels_config.matrix]
homeserver = "https://matrix.example.com"
access_token = "syt_your-access-token"
user_id = "@zeroclaw:matrix.example.com"     # Optional, recommended for E2EE
device_id = "DEVICEID123"                     # Optional, recommended for E2EE
room_id = "!room:matrix.example.com"           # or #alias:matrix.example.com
allowed_users = ["*"]
```

**Build requirements:**
```bash
cargo build --features channel-matrix
```

### Email

**Config:**
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
```

### IRC

**Config:**
```toml
[channels_config.irc]
server = "irc.libera.chat"
port = 6697
nickname = "zeroclaw-bot"
username = "zeroclaw"                    # Optional
channels = ["#zeroclaw"]
allowed_users = ["*"]
server_password = ""                       # Optional
nickserv_password = ""                      # Optional
sasl_password = ""                         # Optional
verify_tls = true
```

### Webhook Channel

**Config:**
```toml
[channels_config.webhook]
port = 8080
secret = "your-shared-secret"
```

**Usage:**
```bash
# Send message
curl -X POST http://localhost:8080/webhook \
  -H "Content-Type: application/json" \
  -H "X-Webhook-Secret: your-shared-secret" \
  -d '{"message": "Hello, ZeroClaw!"}'
```

### DingTalk

**Quick setup:**
```bash
zeroclaw onboard
# Follow prompts for DingTalk
```

**Config:**
```toml
[channels_config.dingtalk]
client_id = "your-app-key"
client_secret = "your-app-secret"
allowed_users = ["*"]
```

### Lark

**Config:**
```toml
[channels_config.lark]
app_id = "cli_xxx"
app_secret = "xxx"
encrypt_key = ""                          # Optional
verification_token = ""                     # Optional
allowed_users = ["*"]
mention_only = false                       # Require @mention in groups
use_feishu = false
receive_mode = "websocket"          # or "webhook"
port = 8081                              # Required for webhook mode
```

**Build requirements:**
```bash
cargo build --features channel-lark
```

### Nostr

**Config:**
```toml
[channels_config.nostr]
private_key = "nsec1..."                     # hex or nsec bech32
allowed_pubkeys = ["hex-or-npub"]          # Empty = deny all, "*" = allow all
```

---

## Validation Workflow

Follow this workflow for any channel setup:

1. **Configure** channel with permissive allowlist (`"*"`) for initial verification
2. **Start** channels:
   ```bash
   zeroclaw channel start
   ```
3. **Test** by sending a message from an expected sender
4. **Confirm** reply arrives
5. **Tighten** allowlist from `"*"` to explicit IDs

```bash
# Interactive onboarding
zeroclaw onboard --channels-only
```

## Troubleshooting

### No Response from Channel

1. **Check allowlist:** Confirm sender identity is allowed
2. **Check permissions:** Verify bot account membership/permissions in room/channel
3. **Verify credentials:** Confirm tokens/secrets are valid and not expired
4. **Check transport mode:**
   - Polling/WebSocket channels: No public HTTP needed
   - Webhook channels: Public HTTPS callback required
5. **Restart daemon:**
   ```bash
   zeroclaw daemon --restart
   ```

### Log Analysis

**Capture logs:**
```bash
RUST_LOG=info zeroclaw daemon 2>&1 | tee /tmp/zeroclaw.log
```

**Filter channel events:**
```bash
rg -n "Telegram|Discord|Slack|WhatsApp" /tmp/zeroclaw.log
```

## Security Best Practices

1. **Never commit tokens** to version control
2. **Use environment variables** for sensitive credentials
3. **Start with `"*"`** allowlist, then tighten
4. **Rotate credentials** regularly
5. **Use webhook secrets** to verify callbacks
6. **Enable TLS** for all network connections
7. **Restrict bot permissions** to minimum required
8. **Audit allowlists** periodically
