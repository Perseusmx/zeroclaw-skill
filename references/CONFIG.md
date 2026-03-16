# ZeroClaw Configuration Reference

ZeroClaw uses `~/.zeroclaw/config.toml` for all configuration settings. This document provides complete reference for all configuration sections.

**Quick setup:**
```bash
# Interactive wizard (recommended)
zeroclaw onboard --interactive

# Quick setup with API key
ZEROCLAW_API_KEY="..." zeroclaw onboard --provider openrouter

# View current config
zeroclaw config get

# Update config values
zeroclaw config set <key> <value>
```

**Config location:** `~/.zeroclaw/config.toml`

**Config resolution order:**
1. `ZEROCLAW_CONFIG_DIR` environment variable
2. `ZEROCLAW_WORKSPACE` environment variable
3. `~/.zeroclaw/active_workspace.toml` marker file
4. Default `~/.zeroclaw/config.toml`

## Core Settings

### Provider Configuration

```toml
# api_key resolved from $ZEROCLAW_API_KEY env var (recommended) or set here (encrypted at rest)
default_provider = "openrouter"
default_model = "anthropic/claude-sonnet-4.5"
default_temperature = 0.7
```

**Settings:**
- `api_key` - Primary API key (encrypted at rest by default)
- `default_provider` - Provider ID (openrouter, anthropic, openai, ollama, etc.)
- `default_model` - Model ID to use by default
- `default_temperature` - Temperature for LLM responses (0.0-2.0)

<security-warning>
**⚠️ SECURITY**: API keys are encrypted at rest when `secrets.encrypt = true` (default). Never commit config.toml to version control.
</security-warning>

### Custom Provider Endpoint

```toml
# OpenAI-compatible endpoint
default_provider = "custom:https://your-api.example.com"

# Anthropic-compatible endpoint
default_provider = "anthropic-custom:https://your-api.example.com"
```

---

## Memory Configuration

```toml
[memory]
backend = "sqlite"              # "sqlite", "postgres", "lucid", "markdown", "none"
auto_save = true
embedding_provider = "openai"     # "openai", "noop"
vector_weight = 0.7
keyword_weight = 0.3
```

**Backend options:**
- `sqlite` - Local SQLite with hybrid search (default, fast, no setup)
- `postgres` - PostgreSQL (configurable, for multi-instance setups)
- `lucid` - Lucid search engine bridge
- `markdown` - Markdown files in workspace
- `none` - No memory backend (stateless)

**Settings:**
- `auto_save` - Automatically save context to memory
- `embedding_provider` - Provider for semantic search
- `vector_weight` - Weight for semantic search results (0.0-1.0)
- `keyword_weight` - Weight for keyword search results (0.0-1.0)

---

## Gateway Configuration

```toml
[gateway]
host = "127.0.0.1"            # Bind address
port = 42617                    # Port to listen on
require_pairing = true            # Require pairing code on first connect
allow_public_bind = false           # Allow binding to 0.0.0.0
pair_rate_limit_per_minute = 10    # Limit pairing attempts
webhook_rate_limit_per_minute = 60
trust_forwarded_headers = false    # Trust X-Forwarded-* headers
```

**Network modes:**
- `host = "127.0.0.1"` - Localhost only (default, secure)
- `host = "0.0.0.0"` - All interfaces (requires `allow_public_bind = true`)

<security-warning>
**⚠️ SECURITY**: Use `allow_public_bind = false` unless behind a firewall. Public binding exposes gateway to entire LAN.
</security-warning>

---

## Autonomy & Security

### Autonomy Levels

```toml
[autonomy]
level = "supervised"            # "supervised", "assisted", "full"
workspace_only = true
allowed_commands = ["git", "npm", "cargo", "ls", "cat", "grep"]
max_actions_per_hour = 20
max_cost_per_day_cents = 500
```

**Autonomy levels:**

| Level | Filesystem | Shell | Approval Required | Use Case |
|---|---|---|---|---|
| `supervised` | Restricted | Allowlisted only | Yes, all actions | Maximum security, production |
| `assisted` | Read/Write | Allowlisted | Risky actions only | Daily use, balance security/usability |
| `full` | Full access | Unrestricted | No | Trusted personal machines only |

<security-warning>
**⚠️ HIGH RISK**: Setting `level = "full"` removes all guardrails. Only use on trusted, personal machines. The AI can execute any command without approval.
</security-warning>

**Security settings:**

```toml
[autonomy]
workspace_only = true              # Restrict to workspace only
allowed_roots = []                # Additional allowed paths
forbidden_paths = ["/etc", "/root", "/proc", "/sys", "~/.ssh"]
blocked_commands = ["rm -rf", "dd", "mkfs"]
```

- `workspace_only` - Restrict filesystem to workspace directory
- `allowed_roots` - Additional paths to allow beyond workspace
- `forbidden_paths` - Paths to block explicitly
- `blocked_commands` - Dangerous commands to block

### Secrets Management

```toml
[secrets]
encrypt = true                    # Encrypt secrets at rest (default: true)
key_derivation = "argon2id"     # Key derivation algorithm
iterations = 100000               # Argon2 iterations
memory_mb = 64                   # Memory limit (MB)
parallelism = 4                   # Parallelism factor
salt = ""                        # Optional custom salt
```

<security-warning>
**⚠️ SECURITY**: Never disable encryption. Secrets in config.toml are automatically encrypted with strong Argon2id when `encrypt = true`.
</security-warning>

---

## Agent Configuration

```toml
[agent]
compact_context = false            # Load full workspace (recommended)
max_tool_iterations = 50          # Max tool calls per task
max_history_messages = 100         # Conversation history length
parallel_tools = true             # Parallel tool execution
tool_dispatcher = "auto"          # "auto", "native", "python"
```

**Settings:**
- `compact_context` - If true, uses compact workspace context
- `max_tool_iterations` - Maximum number of tool calls per task (default: 10)
- `max_history_messages` - Number of conversation messages to retain
- `parallel_tools` - Execute tools in parallel for speed
- `tool_dispatcher` - Method for tool execution

---

## Browser Control

```toml
[browser]
enabled = false
backend = "agent_browser"        # "agent_browser", "rust_native", "computer_use", "auto"
allowed_domains = ["example.com"]
session_name = ""                # Optional session persistence
```

**Backends:**
- `agent_browser` (default) - Calls `agent-browser` CLI; supports accessibility tree snapshots with `@ref` handles
- `rust_native` - In-process WebDriver via fantoccini; requires ChromeDriver; no snapshot support
- `computer_use` - HTTP POST to sidecar for OS-level mouse/keyboard/screen control
- `auto` - Auto-detects best available backend

**Settings:**
- `enabled` - Enable browser automation tools
- `allowed_domains` - Restrict to specific domains (`["*"]` for all)

<security-warning>
**⚠️ SECURITY**: Browser automation can access any website. Restrict with `allowed_domains` in production.
</security-warning>

---

## HTTP Requests

```toml
[http_request]
enabled = false
allowed_domains = []             # ["*"] for all
max_response_size = 0            # 0 = unlimited
timeout_secs = 30
```

**Settings:**
- `enabled` - Enable HTTP request tool
- `allowed_domains` - Domain allowlist
- `max_response_size` - Max response bytes (0 = unlimited)
- `timeout_secs` - Request timeout

---

## Runtime Configuration

### Native Runtime (Default)

```toml
[runtime]
kind = "native"                # "native" or "docker"
```

### Docker Runtime

```toml
[runtime]
kind = "docker"

[runtime.docker]
image = "alpine:3.20"
memory_limit_mb = 512
cpu_limit = 1.0
read_only_rootfs = true
mount_workspace = true
network = "none"               # "none", "bridge", "host"
```

<security-warning>
**⚠️ SECURITY**: Docker runtime provides strong isolation. Always use `read_only_rootfs = true` in production.
</security-warning>

---

## Scheduling & Automation

```toml
[scheduling]
enabled = true
max_concurrent_tasks = 3
task_timeout_secs = 300
```

---

## Cron Jobs

```toml
[[cron_jobs]]
name = "daily-report"
schedule = "0 9 * * *"     # Cron expression
command = "generate daily report"
enabled = true

[[cron_jobs]]
name = "backup"
schedule = "0 2 * * 0"      # Every Sunday at 2 AM
command = "backup workspace"
enabled = true
```

**Manage cron jobs:**
```bash
zeroclaw cron list
zeroclaw cron add <name> <schedule> <command>
zeroclaw cron remove <name>
zeroclaw cron run <name>
```

---

## Response Cache

```toml
[cache]
enabled = true
ttl_secs = 300                    # Cache entry time-to-live
max_entries = 10000               # Max cached responses
analytics = true                  # Enable cache hit/miss analytics
```

**Settings:**
- `enabled` - Enable two-tier response cache
- `ttl_secs` - Time-to-live for cached entries
- `max_entries` - Maximum number of cached responses
- `analytics` - Track cache hit/miss rates and per-provider token usage

---

## Transcription Configuration

```toml
[transcription]
enabled = false
provider = "openai"               # "openai", "google", "local"
model = "whisper-1"
initial_prompt = ""               # Hint for proper nouns, acronyms, jargon
language = ""                     # ISO 639-1 code (e.g., "en")
```

**Settings:**
- `initial_prompt` - Text hint to improve transcription of domain-specific terms
- `language` - Force language detection to specific language

---

## Heartbeat Configuration

```toml
[heartbeat]
enabled = true
interval_secs = 60
adaptive = true                   # Adjust interval based on activity
endpoint = "https://your-server/heartbeat"
include_health = true             # Include health metrics in heartbeat
include_task_history = false      # Include recent task history
```

**Settings:**
- `enabled` - Enable heartbeat monitoring
- `interval_secs` - Base interval for heartbeat (adaptive mode adjusts this)
- `adaptive` - Dynamically adjust interval based on agent activity
- `endpoint` - URL to send heartbeat to
- `include_health` - Attach health metrics to heartbeat payload
- `include_task_history` - Include recent task execution history

---

## Tunnel Configuration

```toml
[tunnel]
provider = "none"                # "none", "cloudflare", "tailscale", "ngrok", "custom"

[tunnel.cloudflare]
account_id = "..."     # from Cloudflare dashboard
api_token = "..."      # from Cloudflare API tokens
tunnel_name = "zeroclaw"

[tunnel.tailscale]
auth_key = "..."       # from Tailscale admin console
tunnel_name = "zeroclaw"

[tunnel.ngrok]
authtoken = "..."      # from ngrok dashboard
domain = "..."         # your ngrok domain

[tunnel.custom]
url = "https://your-custom-tunnel.com"
```

---

## Web Search Configuration

```toml
[web_search]
enabled = true
provider = "duckduckgo"       # "duckduckgo" or "brave"
max_results = 10
timeout_secs = 10
```

<security-warning>
**⚠️ SECURITY**: Web search may access arbitrary URLs. Consider enabling in supervised mode only.
</security-warning>

---

## Cost Limits

```toml
[cost]
enabled = true
max_cost_per_day_cents = 500
max_cost_per_hour_cents = 100
alert_threshold_cents = 80
```

**Settings:**
- `enabled` - Enable cost tracking
- `max_cost_per_day_cents` - Daily spending limit
- `max_cost_per_hour_cents` - Hourly spending limit
- `alert_threshold_cents` - Alert when approaching limit

---

## Logging Configuration

```toml
[logging]
level = "info"                   # "error", "warn", "info", "debug", "trace"
file = "~/.zeroclaw/logs/zeroclaw.log"
max_size_mb = 10
max_files = 5
```

**Log levels:**
- `error` - Critical errors only
- `warn` - Warnings and errors
- `info` - General information (default)
- `debug` - Detailed debugging
- `trace` - Very verbose tracing

---

## Observability & Tracing

```toml
[observability]
tracing_enabled = true
trace_file = "~/.zeroclaw/logs/runtime_trace.jsonl"
opentelemetry_endpoint = ""      # Optional: OpenTelemetry collector URL
```

**Settings:**
- `tracing_enabled` - Enable runtime event tracing
- `trace_file` - Path for JSONL trace output (all tool executions, actions)
- `opentelemetry_endpoint` - Optional collector for distributed tracing

---

## Model Routing

```toml
[[model_routes]]
hint = "reasoning"
provider = "openrouter"
model = "anthropic/claude-opus-4-20250514"

[[model_routes]]
hint = "fast"
provider = "groq"
model = "llama-3.3-70b-versatile"

[[model_routes]]
hint = "code"
provider = "openai"
model = "gpt-4-turbo"
```

**Use hints in tool calls:**
```text
hint:reasoning
hint:fast
hint:code
```

---

## Embedding Routing

```toml
[memory]
embedding_model = "hint:semantic"

[[embedding_routes]]
hint = "semantic"
provider = "openai"
model = "text-embedding-3-small"
dimensions = 1536

[[embedding_routes]]
hint = "archive"
provider = "custom:https://embed.example.com/v1"
model = "your-embedding-model-id"
dimensions = 1024
# api_key from env var or set here (encrypted at rest)
```

---

## Environment Variables

ZeroClaw respects these environment variables:

| Variable | Purpose |
|---|---|
| `API_KEY` | Default API key (fallback) |
| `ZEROCLAW_API_KEY` | ZeroClaw-specific API key (fallback) |
| `HOST` | Gateway host override |
| `PORT` | Gateway port override |
| `PROVIDER` | Default provider (legacy) |
| `MODEL` | Default model override |
| `TEMPERATURE` | Default temperature override |
| `ZEROCLAW_CONFIG_DIR` | Custom config directory (highest priority) |
| `ZEROCLAW_CONFIG_PATH` | Custom config file path |
| `ZEROCLAW_WORKSPACE_DIR` | Custom workspace directory |

**Priority:**
1. Explicit config file value
2. Provider-specific environment variable
3. Generic fallback variables (`ZEROCLAW_API_KEY`, `API_KEY`)

---

## Configuration Validation

Run validation to check configuration:

```bash
# Full validation
zeroclaw doctor

# Check specific config values
zeroclaw config get <key>
```

**Common validation issues:**

| Issue | Solution |
|---|---|
| API key not found | Set `api_key` or environment variable |
| Provider not found | Check `default_provider` ID matches supported providers |
| Model not found | Run `zeroclaw models refresh` |
| Invalid config path | Set `ZEROCLAW_CONFIG_PATH` or use default location |
| Permission denied | Check file permissions: `chmod 600 ~/.zeroclaw/config.toml` |

---

## Best Practices

1. **Use interactive onboarding** for initial setup: `zeroclaw onboard --interactive`
2. **Restrict autonomy** to `supervised` for daily use
3. **Enable workspace_only** to restrict filesystem access
4. **Set cost limits** to prevent overspending
5. **Encrypt secrets** (enabled by default)
6. **Validate config** before daemon start: `zeroclaw doctor`
7. **Review logs** for issues: `RUST_LOG=debug zeroclaw daemon`
8. **Backup config** before major changes: `cp ~/.zeroclaw/config.toml ~/.zeroclaw/config.toml.bak`

---

## Security Checklist

Before deploying to production, verify:

- [ ] Autonomy level is `supervised` or `assisted`
- [ ] `workspace_only = true` unless explicitly needed
- [ ] `secrets.encrypt = true` (default)
- [ ] Cost limits configured
- [ ] Dangerous commands blocked
- [ ] Forbidden paths configured
- [ ] Gateway not public (`allow_public_bind = false`)
- [ ] Tunneling disabled or secured
- [ ] Browser automation restricted (`allowed_domains`)
- [ ] Logs configured for monitoring

<security-warning>
**⚠️ SECURITY**: Review this checklist before any production deployment.
</security-warning>

---

## Reference Documentation

- [SKILL.md](SKILL.md) - Quick reference and essential commands
- [PROVIDERS.md](PROVIDERS.md) - Complete provider catalog
- [CHANNELS.md](CHANNELS.md) - Channel-by-channel setup
- [ZeroClaw Documentation](https://github.com/zeroclaw-labs/zeroclaw)
