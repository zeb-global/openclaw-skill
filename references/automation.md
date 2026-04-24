# Automation: Cron, Webhooks, Hooks

## Cron — Scheduled Tasks

Jobs persist at `~/.openclaw/cron/jobs.json`. Track in git. Gitignore `jobs-state.json`.

```bash
# One-shot reminder (ISO or relative time)
openclaw cron add \
  --name "Reminder" \
  --at "2026-06-01T09:00:00Z" \
  --session main \
  --system-event "Check Q2 review" \
  --wake now \
  --delete-after-run

# Recurring cron expression
openclaw cron add \
  --name "Daily standup" \
  --cron "0 9 * * 1-5" \
  --tz "America/New_York" \
  --session isolated \
  --message "Generate standup summary and post to Slack #standup"

# Fixed interval
openclaw cron add --name "Hourly check" --every "1h" --session isolated

# Manage
openclaw cron list
openclaw cron show <job-id>
openclaw cron runs --id <job-id>
openclaw cron delete <job-id>
```

**Schedule types:**
| Kind | Flag | Notes |
|---|---|---|
| One-shot | `--at` | ISO 8601 or relative (`20m`, `2h`) |
| Fixed interval | `--every` | `1h`, `30m`, `1d` |
| Cron expression | `--cron` | 5- or 6-field; use `--tz` for local time |

**Execution styles:**
| Session | Best for |
|---|---|
| `main` | Reminders, system events; uses next heartbeat |
| `isolated` | Reports, background work; fresh session |
| `current` | Context-aware recurring work |
| `session:<id>` | Workflows that build on persistent history |

**Stagger:** top-of-hour expressions auto-stagger up to 5 min. Use `--exact` to disable. Use `--stagger 30s` for explicit window.

**Day-of-month + day-of-week = OR logic** (Vixie cron standard). `0 9 15 * 1` fires on 15th AND every Monday. Use `+` modifier for AND.

## Webhooks

```json
{
  "plugins": {
    "entries": {
      "webhooks": {
        "enabled": true,
        "routes": [
          { "path": "/hook/deploy", "agent": "main", "message": "Deploy event: {{payload}}" }
        ]
      }
    }
  }
}
```

**Built-in endpoints (all require auth token):**
- `POST /hooks/wake` — wake agent heartbeat
- `POST /hooks/agent` — trigger agent run
- `POST /hooks/<n>` — mapped route per config

## Gmail Pub/Sub

```json
{
  "plugins": {
    "entries": {
      "gmail-pubsub": {
        "enabled": true,
        "agent": "main",
        "model": "bedrock/anthropic.claude-haiku-4-5"
      }
    }
  }
}
```

Set up via `openclaw onboard` (recommended). Triggers agent run on new email.

## Hooks (Gateway Event Triggers)

```json
{
  "automation": {
    "hooks": [
      {
        "event": "session.start",
        "message": "New session. Load AGENTS.md context.",
        "agent": "main"
      }
    ]
  }
}
```

Events: `session.start`, `session.end`, `message.received`, gateway lifecycle.

## Standing Orders (AGENTS.md)

Persistent cross-session instructions belong in `AGENTS.md`:
```markdown
# Standing Orders

- Check project checklist before starting new tasks
- Review code with diff tool; comment inline
- All times in US Eastern
- Morning: load memory/today.md and summarize overnight
```
