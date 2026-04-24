# Use Case Playbooks

## Personal Executive Assistant (Slack + WhatsApp)

**Config:**
```json
{
  "agent": { "model": "bedrock/anthropic.claude-sonnet-4-5" },
  "channels": {
    "slack": { "appToken": "...", "botToken": "...", "dmPolicy": "pairing", "allowFrom": ["UYOURID"] },
    "whatsapp": { "allowFrom": ["+1yourphone"] }
  }
}
```

**Skills to write:** `daily-brief` (summarize memory + agenda), `task-capture` (write todos to AGENTS.md), `research` (web_search + web_fetch workflow).

**AGENTS.md:** standing orders for morning summary format, todo style, response tone.

**Automation:** daily cron at 8am, isolated session, message to Slack with overnight summary.

---

## Team Knowledge Bot (Slack, read-only)

```json
{
  "agents": {
    "list": [{
      "id": "knowledge",
      "tools": { "profile": "minimal", "allow": ["web_fetch", "read"] },
      "skills": ["docs-search", "faq"],
      "channels": { "slack": { "workspaces": ["TTEAMID"], "dmPolicy": "open", "allowFrom": ["*"] } }
    }]
  }
}
```

**Skills:** `docs-search` (web_fetch against internal doc URLs), `faq` (hardcoded top questions).

---

## Automated Daily Report (Cron → Slack)

```bash
openclaw cron add \
  --name "Daily Report" \
  --cron "0 8 * * 1-5" --tz "America/Chicago" \
  --session isolated \
  --message "Read ~/data/metrics.csv, summarize yesterday's numbers, post to Slack #reports"
```

**Skill to write:** `daily-report` — read CSV, format summary, call `message` tool to Slack.

---

## GitHub Webhook → Agent Action

```json
{
  "plugins": {
    "entries": {
      "webhooks": {
        "enabled": true,
        "routes": [{ "path": "/hook/github", "agent": "main", "message": "GitHub event: {{payload}}" }]
      }
    }
  }
}
```

Point GitHub webhook to `https://your-host/hooks/github` with auth header.
**Skill:** `github-events` — interpret PR/issue payload, decide action.

---

## Multi-Agent Research Pipeline

```json
{
  "agents": {
    "defaults": { "subagents": { "model": "bedrock/anthropic.claude-haiku-4-5" } }
  }
}
```

Flow: main agent receives research request → spawns 3 sub-agents in parallel via `sessions_spawn` with `runTimeoutSeconds: 120` → each does web_search + web_fetch + summary → each announces result → main synthesizes and delivers.

---

## Internal IT Helpdesk Bot (Teams)

```json
{
  "agents": {
    "list": [{
      "id": "helpdesk",
      "tools": { "profile": "messaging", "allow": ["web_search", "read"] },
      "channels": { "teams": { "tenantId": "...", "botId": "...", "dmPolicy": "open", "allowFrom": ["*"] } }
    }]
  }
}
```

**Skills:** `it-faq` (top IT questions + KB URLs), `ticket-triage` (classify + route to team).

---

## Voice + Text Unified Assistant (Nova Sonic + OpenClaw)

Architecture:
- OpenClaw Gateway on Lightsail → handles WhatsApp/Slack/Telegram
- Amazon Connect + Nova Sonic → handles inbound voice calls
- Shared Bedrock knowledge base or S3 session store
- Webhook plugin routes Connect events to OpenClaw agent

**Skill:** `connect-handoff` — read Connect session context, continue conversation from voice to text.

---

## Secure Enterprise Assistant (VPC, compliance-first)

```json
{
  "gateway": { "bind": "10.0.1.5", "auth": { "token": "..." } },
  "agents": {
    "defaults": {
      "model": "bedrock/anthropic.claude-sonnet-4-5",
      "sandbox": { "mode": "non-main" },
      "tools": { "deny": ["browser", "exec"] }
    }
  },
  "logging": { "redactSensitive": "tools", "retention": "30d" }
}
```

**Add:** VPC endpoints for Bedrock, CloudTrail logging, GuardDuty, Config rules.
