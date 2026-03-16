# ZeroClaw Security Best Practices

Security guidance for operating ZeroClaw autonomous AI infrastructure. For config details see [CONFIG.md](CONFIG.md), for channel allowlists see [CHANNELS.md](CHANNELS.md).

**Quick security check:**
```bash
zeroclaw doctor
zeroclaw config get autonomy
zeroclaw config get gateway
```

---

## Security Model

ZeroClaw implements **defense-in-depth** across seven enforcement layers:

1. **Autonomy level gating** - Initial scope check (supervised/assisted/full)
2. **Emergency stop (E-Stop)** - Multi-granularity shutdown state machine (`kill_all`, `network_kill`, `domain_block`, `tool_freeze`)
3. **OTP validation** - Time-based one-time codes for sensitive operations
4. **Command allowlisting** - Explicit permit/deny with risk classification (high/medium/low)
5. **Path workspace scoping** - Canonicalized path validation with symlink detection
6. **Rate limiting** - Action throttling via ActionTracker per configured time window
7. **Domain validation** - Browser navigation and HTTP request restrictions

**Encryption:** ChaCha20-Poly1305 AEAD for secrets at rest, with Argon2id key derivation.

<security-warning>
**⚠️ CRITICAL**: ZeroClaw gives an autonomous AI powerful capabilities. Always verify security settings before running in production.
</security-warning>

---

## Pre-Deployment Checklist

### Autonomy and Access
- [ ] Autonomy level is `supervised` or `assisted` (not `full`)
- [ ] `workspace_only = true` unless explicitly needed
- [ ] `allowed_commands` explicitly lists safe commands
- [ ] `blocked_commands` includes `rm -rf`, `dd`, `mkfs`, `fdisk`, `cryptsetup`
- [ ] `forbidden_paths` blocks `/etc`, `/root`, `/proc`, `/sys`, `~/.ssh`, `~/.gnupg`, `~/.aws`
- [ ] Cost limits configured (`max_cost_per_day_cents`, `max_actions_per_hour`)

### Secrets and Credentials
- [ ] `secrets.encrypt = true` (default, verify)
- [ ] No API keys in version control
- [ ] No credentials in shell history
- [ ] Environment variables used for sensitive data
- [ ] File permissions set: `chmod 600 ~/.zeroclaw/config.toml`
- [ ] Key rotation schedule defined

### Gateway and Channels
- [ ] Gateway `allow_public_bind = false` (default, verify)
- [ ] Gateway `require_pairing = true`
- [ ] All channels use explicit allowlists (not `"*"` after verification)
- [ ] Webhook verify tokens / signing secrets configured
- [ ] Rate limits configured

### Runtime and Network
- [ ] Docker runtime used with `read_only_rootfs = true` (if applicable)
- [ ] Sandbox backend configured (Landlock, Firejail, or Bubblewrap) if needed
- [ ] Browser automation restricted with `allowed_domains`
- [ ] Tunnel provider is secure (Cloudflare/Tailscale preferred over ngrok)
- [ ] Logging configured with rotation (`max_size_mb`, `max_files`)

---

## Credential Resolution Order

1. Explicit value in `config.toml`
2. Provider-specific environment variable (e.g. `ANTHROPIC_API_KEY`)
3. `ZEROCLAW_API_KEY` environment variable
4. `API_KEY` environment variable (generic fallback)

Auth profiles stored in `~/.zeroclaw/auth-profiles.json` (encrypted with `~/.zeroclaw/.secret_key`).

---

## Common Security Mistakes

### Autonomy
- Setting `level = "full"` in untrusted environments
- Using `assisted` when `supervised` is appropriate for production
- Disabling `workspace_only` without need
- Allowing all commands without explicit allowlist
- Omitting cost limits in production

### Secrets
- Committing `api_key` to version control
- Storing credentials in plaintext (disable `secrets.encrypt`)
- Sharing API keys via chat or email
- Never rotating credentials

### Channels & Gateway
- Keeping `"*"` allowlist in production
- Setting `allow_public_bind = true` without firewall
- Disabling pairing requirement
- Disabling webhook verification
- Using ngrok in production (prefer Cloudflare/Tailscale)

### Runtime
- Using native runtime without sandbox in production
- Disabling `read_only_rootfs` in Docker
- Enabling browser without `allowed_domains`
- Not using available sandbox backends (Landlock on Linux 5.13+, Firejail, or Bubblewrap)

---

## Incident Response

**If security incident suspected:**

1. **Contain immediately:**
   - `zeroclaw service stop`
   - Disable affected channels
   - Rotate compromised credentials

2. **Investigate:**
   - Review logs: `journalctl --user -u zeroclaw.service` or `~/.zeroclaw/logs/`
   - Check audit log for suspicious events
   - Review cost logs for unexpected charges

3. **Recover:**
   - Rotate all credentials
   - Tighten allowlists
   - Verify autonomy level
   - `zeroclaw service start`

---

## Regular Maintenance

**Weekly:** Review cost logs, check blocked command patterns, verify allowlists

**Monthly:** Full security config review, update blocked commands, audit channel access, review tunnel/proxy config

**Quarterly:** Rotate all credentials, security policy review, consider penetration testing

---

## Secure Configuration Examples

### Development

```toml
[autonomy]
level = "supervised"
workspace_only = true
allowed_commands = ["git", "npm", "cargo", "ls", "cat", "grep", "sed"]
max_actions_per_hour = 50
max_cost_per_day_cents = 1000

[gateway]
host = "127.0.0.1"
port = 42617
require_pairing = true
allow_public_bind = false

[secrets]
encrypt = true
```

### Production

```toml
[autonomy]
level = "supervised"
workspace_only = true
allowed_commands = ["git", "cargo", "ls", "cat"]
forbidden_paths = ["/etc", "/root", "/proc", "/sys", "~/.ssh"]
blocked_commands = ["rm -rf", "dd", "mkfs"]
max_actions_per_hour = 20
max_cost_per_day_cents = 500

[gateway]
host = "127.0.0.1"
port = 42617
require_pairing = true
allow_public_bind = false
pair_rate_limit_per_minute = 5
webhook_rate_limit_per_minute = 30

[secrets]
encrypt = true
key_derivation = "argon2id"
iterations = 200000

[logging]
level = "warn"
file = "~/.zeroclaw/logs/zeroclaw.log"
max_size_mb = 50
max_files = 10

[browser]
enabled = false
```

### Multi-Tenant / Shared System

```toml
[autonomy]
level = "supervised"
workspace_only = true
max_actions_per_hour = 10

[secrets]
encrypt = true
```
