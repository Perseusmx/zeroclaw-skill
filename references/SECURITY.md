# ZeroClaw Security Best Practices

This reference was refreshed against the current Zeroclaw docs and CLI surface on **March 28, 2026**.

## Quick Security Check

```bash
zeroclaw doctor
zeroclaw config get autonomy
zeroclaw config get gateway
zeroclaw estop status
```

## Security Model

ZeroClaw layers safety controls across:
1. Autonomy level gating
2. Emergency stop controls (`kill-all`, `network-kill`, `domain-block`, `tool-freeze`)
3. Approval and allowlist enforcement
4. Command policy and blocked-command checks
5. Workspace/path scoping
6. Rate and cost limiting
7. Domain restrictions for browser and HTTP tools

Secrets are encrypted at rest by default with Argon2id-based key derivation.

## Deployment Checklist

### Autonomy
- [ ] `level` is `supervised` or `assisted`
- [ ] `workspace_only = true` unless there is a clear exception
- [ ] `allowed_commands` is intentionally small
- [ ] `blocked_commands` includes destructive disk and wipe commands
- [ ] `forbidden_paths` covers system and credential directories
- [ ] Action and cost limits are set

### Secrets
- [ ] `secrets.encrypt = true`
- [ ] No API keys are committed
- [ ] Sensitive env vars are used instead of plaintext config where possible
- [ ] `~/.zeroclaw/config.toml` has restrictive permissions

### Gateway and Channels
- [ ] `allow_public_bind = false` unless exposure is intentional
- [ ] Pairing is required
- [ ] Channel allowlists are explicit, not `"*"`
- [ ] Webhook signing secrets are configured where supported

### Tooling and Network
- [ ] Browser automation is restricted with `allowed_domains`
- [ ] HTTP tool domains are allowlisted
- [ ] Docker isolation uses `read_only_rootfs = true` if Docker runtime is enabled
- [ ] `zeroclaw estop` is tested before production rollout

## Common Mistakes

- Running with `level = "full"` on shared or untrusted machines
- Leaving `"*"` in a channel allowlist after testing
- Exposing the gateway on `0.0.0.0` without a strict network boundary
- Giving browser or HTTP tools unrestricted domains
- Treating `assisted` as production-safe without reviewing allowed commands

## Incident Response

Contain:

```bash
zeroclaw estop
zeroclaw service stop
```

Investigate:
- Review `journalctl --user -u zeroclaw.service`
- Review `~/.zeroclaw/logs/`
- Review recent config changes and provider usage

Recover:
- Rotate credentials
- Tighten channel allowlists
- Re-check `[autonomy]`, `[gateway]`, and domain restrictions
- Resume only after validation with `zeroclaw estop status`

## Recommended Baseline

```toml
[autonomy]
level = "supervised"
workspace_only = true
allowed_commands = ["git", "cargo", "ls", "cat"]
blocked_commands = ["rm -rf", "dd", "mkfs", "fdisk", "cryptsetup"]
max_actions_per_hour = 20
max_cost_per_day_cents = 500

[gateway]
host = "127.0.0.1"
port = 42617
require_pairing = true
allow_public_bind = false

[secrets]
encrypt = true

[browser]
enabled = false
```
