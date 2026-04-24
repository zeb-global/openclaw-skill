# Security and Hardening

## Trust Model

One trusted operator boundary per gateway. NOT a multi-tenant hostile-user system. If multiple untrusted users share one agent, they share its tool authority.

**Core principle:** access control before intelligence. Allowlists and sandbox policies enforce at the gateway layer — the model cannot override them.

## Quick Audit

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix        # auto-remediates common issues
openclaw security audit --json
```

Run after every config change. Run after every update.

## Hardened Baseline

```json
{
  "gateway": {
    "bind": "127.0.0.1",
    "auth": { "token": "generate-a-long-random-token-here" }
  },
  "logging": { "redactSensitive": "tools" },
  "agents": {
    "defaults": {
      "sandbox": { "mode": "non-main" }
    }
  }
}
```

- `bind: "127.0.0.1"` — not on network interfaces; use Tailscale or SSH tunnel for remote
- `auth.token` — all WebSocket connections require the token
- `redactSensitive: "tools"` — strips credentials from logs
- `sandbox.mode: "non-main"` — non-main sessions run in Docker sandboxes

**File permissions:**
```bash
chmod 700 ~/.openclaw
chmod 600 ~/.openclaw/openclaw.json
```

## DM Hardening

Use `pairing` + explicit `allowFrom`. Never `allowFrom: ["*"]` with `dmPolicy: "open"` unless fully trusted. Separate credentials for public bots.

## Sandbox Config

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",
        "docker": {
          "allow": ["bash", "process", "read", "write", "edit", "sessions_list", "sessions_history", "sessions_send"],
          "deny": ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
          "setupCommand": "apt-get install -y ffmpeg"
        }
      }
    }
  }
}
```

`setupCommand` runs once after container creation.

## Auth Modes

- `shared-secret` (default) — token in `connect.params.auth.token`
- `trusted-proxy` + Tailscale Serve — auth from request headers
- `none` — disables shared-secret; ONLY for private-ingress setups, never public

## Gateway WebSocket Auth

```
connect.params.auth.token = <your-token>
# or
connect.params.auth.password = <your-password>
```

Idempotency keys required for `send` and `agent` methods to safely retry.

## Credential Map

| What | Location |
|---|---|
| Channel tokens | `~/.openclaw/openclaw.json` |
| Model API keys | `~/.openclaw/openclaw.json` or env |
| Skill API keys | `skills.entries.<n>.apiKey` |
| OAuth / auth profiles | `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` |
| Session transcripts | `~/.openclaw/agents/<agentId>/sessions/` |
| Channel credentials | `~/.openclaw/credentials/` |

Do NOT commit any of these to the workspace git repo.

## Prompt Injection

External content is always untrusted. OpenClaw sanitizes special tokens. Do not use `--allow-unsafe-external-content` or `--disable-token-sanitization`.

## Incident Response

```bash
openclaw gateway stop
# Rotate ALL exposed credentials (assume compromise if any leak)
openclaw logs --session <id> --full
openclaw security audit --json > audit-$(date +%Y%m%d).json
```
