# ZeroClaw Configuration Reference

This reference was refreshed against the official operator-oriented config docs on **March 28, 2026**.

## Quick Commands

```bash
zeroclaw config show
zeroclaw config get gateway.port
zeroclaw config set default_provider openrouter
zeroclaw config schema
```

## Config Resolution Order

At startup, ZeroClaw resolves config from:
1. `ZEROCLAW_WORKSPACE`
2. `~/.zeroclaw/active_workspace.toml`
3. Default `~/.zeroclaw/config.toml`

ZeroClaw logs the resolved config path and source at startup.

## Core Keys

```toml
default_provider = "openrouter"
provider_api = "openai-chat-completions" # optional for custom providers
default_model = "anthropic/claude-sonnet-4-6"
default_temperature = 0.7
model_support_vision = true              # optional override
```

Notes:
- `model_support_vision` can force-enable or force-disable multimodal handling.
- Environment overrides exist for some fields, including `ZEROCLAW_MODEL_SUPPORT_VISION`.

## Named Provider Profiles

```toml
default_provider = "sub2api"

[model_providers.sub2api]
name = "sub2api"
base_url = "https://api.example.com/v1"
wire_api = "chat_completions" # or "responses"
auth_header = "Authorization" # optional custom header
model = "qwen-max"
api_key = "sk-profile-key"
requires_openai_auth = false
```

Use `[model_providers.<name>]` when you need multiple custom or profile-scoped endpoints.

## Observability

```toml
[observability]
backend = "otel" # none | log | prometheus | otel
otel_endpoint = "http://localhost:4318"
otel_service_name = "zeroclaw"
runtime_trace_mode = "rolling" # none | rolling | full
runtime_trace_path = "state/runtime-trace.jsonl"
runtime_trace_max_entries = 200
```

Helpful commands:

```bash
zeroclaw doctor traces --limit 20
zeroclaw doctor traces --event tool_call_result --contains "error"
```

## Autonomy and Safety

```toml
[autonomy]
level = "supervised" # supervised | assisted | full
workspace_only = true
allowed_commands = ["git", "npm", "cargo", "ls", "cat"]
allowed_roots = []
forbidden_paths = ["/etc", "/root", "/proc", "/sys", "~/.ssh"]
blocked_commands = ["rm -rf", "dd", "mkfs"]
max_actions_per_hour = 20
max_cost_per_day_cents = 500
```

Level guidance:
- `supervised`: approval for risky actions; safest default
- `assisted`: allowlisted automation with moderate oversight
- `full`: no approval gating; trusted systems only

## Gateway

```toml
[gateway]
host = "127.0.0.1"
port = 42617
require_pairing = true
allow_public_bind = false
pair_rate_limit_per_minute = 10
webhook_rate_limit_per_minute = 60
trust_forwarded_headers = false
```

Use `allow_public_bind = false` unless you explicitly need LAN exposure.

## Secrets

```toml
[secrets]
encrypt = true
key_derivation = "argon2id"
iterations = 100000
memory_mb = 64
parallelism = 4
```

Secrets are encrypted at rest by default. Do not commit `config.toml`.

## Agent and Runtime

```toml
[agent]
compact_context = false
max_tool_iterations = 50
max_history_messages = 100
parallel_tools = true
tool_dispatcher = "auto"

[runtime]
kind = "native" # or "docker"
reasoning_enabled = false # optional, used by Ollama integrations
```

Docker mode:

```toml
[runtime]
kind = "docker"

[runtime.docker]
image = "alpine:3.20"
memory_limit_mb = 512
cpu_limit = 1.0
read_only_rootfs = true
mount_workspace = true
network = "none"
```

## Browser and HTTP

```toml
[browser]
enabled = false
backend = "agent_browser" # agent_browser | rust_native | computer_use | auto
allowed_domains = ["example.com"]

[http_request]
enabled = false
allowed_domains = []
max_response_size = 0
timeout_secs = 30
```

## E-Stop and Operations

ZeroClaw includes emergency-stop controls for:
- `kill-all`
- `network-kill`
- `domain-block`
- `tool-freeze`

Useful commands:

```bash
zeroclaw estop
zeroclaw estop status
zeroclaw estop resume
zeroclaw estop resume --network
```

## Recommended Baseline

```toml
default_provider = "openrouter"
default_model = "anthropic/claude-sonnet-4-6"

[autonomy]
level = "supervised"
workspace_only = true
allowed_commands = ["git", "cargo", "ls", "cat"]
max_actions_per_hour = 20
max_cost_per_day_cents = 500

[gateway]
host = "127.0.0.1"
port = 42617
require_pairing = true
allow_public_bind = false

[secrets]
encrypt = true
```
