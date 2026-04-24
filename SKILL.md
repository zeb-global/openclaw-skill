---
name: openclaw
description: "The definitive reference for OpenClaw, the self-hosted personal AI assistant gateway. Use for everything OpenClaw - deploy on Linux or AWS Lightsail, configure, write skills, harden security, multi-agent systems, channels (WhatsApp, Telegram, Slack, Discord, Signal, iMessage, Teams, Matrix), cron, webhooks, use case playbooks. Also use when authoring or debugging any AgentSkills SKILL.md file."
---

# OpenClaw — Complete Agent Reference

OpenClaw is a self-hosted, personal AI assistant gateway. It connects to the messaging channels you already use, routes everything through a local Gateway control plane, and gives the model access to real tools: shell, browser, file I/O, sessions, cron, canvas, image/video/music generation, and more. The user owns the host, the credentials, and the model provider relationship entirely.

**Use case speed:** go to the section you need. Every section is self-contained and actionable.

---

## Table of Contents

1. [Architecture](#1-architecture)
2. [Install and First Run](#2-install-and-first-run)
3. [Workspace Files](#3-workspace-files)
4. [Core Concepts](#4-core-concepts)
5. [Channel Setup](#5-channel-setup)
6. [Tools Reference](#6-tools-reference)
7. [Skills System](#7-skills-system)
8. [Writing SKILL.md Files](#8-writing-skillmd)
9. [Security and Hardening](#9-security)
10. [Multi-Agent and Sub-Agent Patterns](#10-multi-agent)
11. [Automation: Cron, Webhooks, Hooks](#11-automation)
12. [AWS Lightsail Deployment](#12-aws-lightsail)
13. [Use Case Playbooks](#13-use-case-playbooks)
14. [Troubleshooting](#14-troubleshooting)
15. [Key URLs](#15-key-urls)

For deep-dive content on specific sections, read the reference files in `references/`.

---

## 1. Architecture

```
External Channels (22 total)
WhatsApp · Telegram · Slack · Discord · Signal · iMessage
Teams · Matrix · IRC · LINE · WeChat · QQ · Twitch · more
         |
         v
  OpenClaw Gateway (daemon · port 18789 · WebSocket API)
  Sessions · Agents · Tools · Cron · Skills · Events
         |
   ------+------
   |            |
Model Provider       Companion Apps (optional)
Bedrock · OpenAI     macOS: menu bar app, push-to-talk
Anthropic · Gemini   iOS / Android: node apps
Ollama · etc.        Linux: headless — gateway only
   |
   v
Built-in Tools
exec/bash · browser · read/write/edit · web_search/web_fetch
image · image_generate · music_generate · video_generate · tts
canvas · nodes · cron · gateway · sessions_* · subagents · message
```

**Key invariants:**
- One Gateway per host. Sole owner of WhatsApp/Telegram/etc. sessions.
- Gateway exposes a typed WebSocket API; first frame must be `connect`.
- Skills = markdown files injected into the system prompt. Teach behavior. Do NOT add tools.
- Tools = typed functions the model calls. Built-in or registered by plugins.
- Plugins = packages that register channels, tools, skills, providers, or media.
- Workspace = agent's only cwd for file tools and context. Default: `~/.openclaw/workspace`
- Session transcripts: `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`
- Config: `~/.openclaw/openclaw.json`
- Workspace skills: `~/.openclaw/workspace/skills/<skill-name>/SKILL.md`

---

## 2. Install and First Run

**Requirements:** Node 24 (recommended) or Node 22.16+. macOS, Linux, or Windows via WSL2.

**Node on Linux (if not already installed):**
```bash
# Option A: nvm (recommended, version-flexible)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
nvm install 24 && nvm use 24

# Option B: NodeSource apt repo
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt-get install -y nodejs
```

```bash
# Install
npm install -g openclaw@latest

# Guided setup (recommended)
# --install-daemon installs as launchd service (macOS) or systemd user service (Linux)
openclaw onboard --install-daemon

# Health check
openclaw doctor

# Test from CLI
openclaw agent --message "Ship checklist" --thinking high
openclaw message send --to +1234567890 --message "Hello"
```

**Add channels at any time:** `openclaw channel add <channel>`

**Update:**
```bash
openclaw update --channel stable     # or: beta, dev
openclaw doctor                      # always run post-update
```

---

## 3. Workspace Files

Default location: `~/.openclaw/workspace`. Every file is user-editable. Large files are trimmed on inject; blank files are skipped.

**Truncation limits:** per file: `bootstrapMaxChars` (default 20000); total: `bootstrapTotalMaxChars` (default 150000).

| File | Purpose | When Loaded |
|---|---|---|
| `AGENTS.md` | Operating rules, priorities, memory, standing orders | Every session |
| `SOUL.md` | Persona, tone, boundaries, opinions | Every session |
| `USER.md` | Who the user is, how to address them | Every session |
| `IDENTITY.md` | Agent name, vibe, emoji | Every session |
| `TOOLS.md` | Tool usage notes and conventions | Every session |
| `HEARTBEAT.md` | Short checklist for periodic heartbeat runs | Every session |
| `BOOT.md` | Startup checklist on gateway restart | Gateway restart |
| `BOOTSTRAP.md` | One-time first-run ritual | New workspace only |
| `memory/YYYY-MM-DD.md` | Daily memory logs | Today + yesterday |
| `MEMORY.md` | Curated long-term memory | Private/main session only |
| `skills/` | Workspace-level skills (highest precedence) | Session start |
| `canvas/` | Canvas UI files for node displays | On canvas tool use |

For SOUL.md guidance and workspace git backup, read [references/workspace-details.md](references/workspace-details.md).

---

## 4. Core Concepts

For full details on sessions, agents, models, queue/steering, and block streaming, read [references/core-concepts.md](references/core-concepts.md).

### Sessions
Each conversation is a session. Isolated by default. Transcripts stored as JSONL.

**Key chat commands:** `/status`, `/new`, `/reset`, `/compact`, `/think <level>`, `/verbose on|off`, `/trace on|off`, `/usage off|tokens|full`, `/restart`, `/activation mention|always`

### Agents
Multiple agents on one gateway, each with own workspace, model, channel routing, skill allowlist, sandbox policy.

### Models
Format: `provider/model`. Split on first `/`. Provider failover is automatic.

---

## 5. Channel Setup

For full channel config examples (Telegram, Slack, Discord, WhatsApp, Teams) and group chat settings, read [references/channels.md](references/channels.md).

### DM Policy (critical)

Default: `dmPolicy: "pairing"`. Unknown senders receive a code; bot ignores until approved.

**Options:**
- `pairing` (default) — unknown senders gated by code
- `open` — all DMs processed (requires `allowFrom: ["*"]`; trusted bots only)
- `disabled` — reject all DMs

### All 22 Channels
WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage (BlueBubbles), iMessage (legacy), IRC, Microsoft Teams, Matrix, Feishu, LINE, Mattermost, Nextcloud Talk, Nostr, Synology Chat, Tlon, Twitch, Zalo, WeChat, QQ, WebChat.

---

## 6. Tools Reference

For full tool tables, tool groups, tool profiles, and plugin-provided tools, read [references/tools.md](references/tools.md).

Three layers: Tools (what the model calls) → Skills (when/how to use them) → Plugins (packages all three).

### Key Built-in Tools
`exec`/`process`, `code_execution`, `browser`, `web_search`/`web_fetch`, `read`/`write`/`edit`, `apply_patch`, `message`, `canvas`, `nodes`, `cron`, `gateway`, `image`/`image_generate`, `music_generate`, `video_generate`, `tts`, `sessions_*`/`subagents`

### Tool Allow/Deny
```json
{
  "tools": {
    "allow": ["group:fs", "browser", "web_search"],
    "deny": ["exec"]
  }
}
```
Deny always beats allow.

---

## 7. Skills System

### What Skills Do (and don't do)
Skills = `SKILL.md` files injected into system prompt. They teach the model when and how to use tools. They do NOT add tools.

### Precedence (highest to lowest)
1. `<workspace>/skills/` — Per-agent (highest)
2. `<workspace>/.agents/skills/` — Per-workspace agent
3. `~/.agents/skills/` — Shared across workspaces
4. `~/.openclaw/skills/` — All agents on machine
5. Bundled (shipped) — Global
6. `skills.load.extraDirs` — Custom shared (lowest)

### Manage
```bash
openclaw skills list
openclaw skills install <slug>        # from ClawHub
openclaw skills update --all
```

**Hot reload:** edit SKILL.md → watcher picks it up on next agent turn (no restart needed).

For per-agent allowlists, token cost, skills config, and Skill Workshop plugin, read [references/skills-system.md](references/skills-system.md).

---

## 8. Writing SKILL.md Files

### Minimum Valid Skill

```markdown
---
name: my-skill
description: One line. Action-oriented. Match real user phrases.
---

# My Skill

When the user asks X, do Y using tool Z.
```

### Full Frontmatter Reference

```yaml
---
name: image-lab
description: "Generate or edit images..."
homepage: https://example.com
user-invocable: true
disable-model-invocation: false
command-dispatch: tool
command-tool: exec
command-arg-mode: raw
metadata: {"openclaw": {"emoji": "🔬", "os": ["darwin", "linux"], "requires": {"bins": ["ffmpeg"], "anyBins": ["convert", "magick"], "env": ["GEMINI_API_KEY"], "config": ["browser.enabled"]}, "primaryEnv": "GEMINI_API_KEY", "always": false, "install": [{"id": "brew", "kind": "brew", "formula": "ffmpeg", "bins": ["ffmpeg"], "label": "Install ffmpeg", "os": ["darwin"]}]}}
---
```

**CRITICAL:** all frontmatter keys must be single-line. `metadata` must be a single-line JSON object. Multi-line YAML blocks break the parser.

### Frontmatter Fields

| Field | Default | Purpose |
|---|---|---|
| `name` | required | Unique identifier, snake_case |
| `description` | required | One line, shown to model — this is the trigger |
| `homepage` | optional | URL shown in Skills UI and ClawHub |
| `user-invocable` | `true` | Exposes as slash command |
| `disable-model-invocation` | `false` | Exclude from prompt; slash-command only |
| `command-dispatch` | optional | `tool` = bypass model, call tool directly |
| `command-tool` | required if dispatch | Tool to invoke |
| `command-arg-mode` | `raw` | How args are passed to tool |
| `metadata.openclaw.emoji` | optional | Emoji for Skills UI and ClawHub |
| `metadata.openclaw.os` | optional | `["darwin"]`, `["linux"]`, `["win32"]` |
| `metadata.openclaw.requires.bins` | optional | ALL must be on PATH |
| `metadata.openclaw.requires.anyBins` | optional | At least ONE must be on PATH |
| `metadata.openclaw.requires.env` | optional | Each must exist in env |
| `metadata.openclaw.requires.config` | optional | Each config path must be truthy |
| `metadata.openclaw.primaryEnv` | optional | Links to `skills.entries.<n>.apiKey` |
| `metadata.openclaw.always` | `false` | Skip all gates, always load |
| `metadata.openclaw.install` | optional | Installer specs for auto-install |

### Body — Writing Rules

**DO:**
- First line: state exactly what the skill does
- Concrete decision rules: "When user says X → use tool Y with params Z"
- Reference files with relative paths: `read references/api.md`
- Numbered steps for multi-step workflows
- Keep body under 500 lines — use reference files for deep content
- Include example invocations

**DON'T:**
- Tell the model how to "be" an AI
- Repeat system prompt content
- Write ambient instructions that fire every turn
- Use vague language ("help with..." → wrong; "run exec with..." → right)

### Reference File Pattern

```
skills/my-skill/
├── SKILL.md
└── references/
    ├── api-schema.md
    └── examples.md
```

### Portability Note

OpenClaw skills follow the open AgentSkills standard. A SKILL.md written for OpenClaw is structurally compatible with other AgentSkills tools. OpenClaw-specific frontmatter is silently ignored by other tools.

---

## 9. Security and Hardening

For full security details including sandbox config, auth modes, credential map, and incident response, read [references/security.md](references/security.md).

### Trust Model
One trusted operator boundary per gateway. NOT a multi-tenant hostile-user system.

### Quick Audit
```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

### Hardened Baseline
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

**File permissions:**
```bash
chmod 700 ~/.openclaw
chmod 600 ~/.openclaw/openclaw.json
```

---

## 10. Multi-Agent and Sub-Agent Patterns

For full multi-agent config, channel routing, sub-agent commands/params, access profiles, and ACP agents, read [references/multi-agent.md](references/multi-agent.md).

### Sub-Agents
Sub-agents = background agent runs spawned from an existing run. Own session, own context, own token budget.

**Key rules:**
- Completion is push-based. Do NOT poll `sessions_history` in a loop.
- Each sub-agent has its OWN context + token usage. Use cheaper model: `agents.defaults.subagents.model`.
- Session ID format: `agent:<agentId>:subagent:<uuid>`

---

## 11. Automation: Cron, Webhooks, Hooks

For full cron examples, webhook config, Gmail Pub/Sub, hooks, and standing orders, read [references/automation.md](references/automation.md).

### Cron — Scheduled Tasks
```bash
# One-shot reminder
openclaw cron add --name "Reminder" --at "2026-06-01T09:00:00Z" --session main --system-event "Check Q2 review" --wake now --delete-after-run

# Recurring
openclaw cron add --name "Daily standup" --cron "0 9 * * 1-5" --tz "America/New_York" --session isolated --message "Generate standup summary"

# Manage
openclaw cron list
openclaw cron delete <job-id>
```

---

## 12. AWS Lightsail Deployment

Lightsail blueprints run **Linux (Ubuntu)**. All commands assume SSH into the instance.

```bash
# Install Node 24 (Ubuntu/Debian)
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install OpenClaw
npm install -g openclaw@latest

# Onboard — installs as systemd user service on Linux
openclaw onboard --install-daemon
openclaw doctor
```

**Verify daemon:**
```bash
systemctl --user status openclaw-gateway
journalctl --user -u openclaw-gateway -f
```

### Bedrock Config
```json
{
  "agent": { "model": "bedrock/anthropic.claude-sonnet-4-5" },
  "agents": { "defaults": { "model": "bedrock/anthropic.claude-sonnet-4-5" } }
}
```

### Lightsail Security Checklist
- Static IP assigned
- HTTPS configured (load balancer + ACM or Let's Encrypt)
- Gateway bound to 127.0.0.1
- IAM role: bedrock:InvokeModel only
- Daily snapshots enabled
- `openclaw doctor` → zero critical findings
- `openclaw security audit --fix` run
- DM policies: pairing, not open
- Auth token: strong random value
- `~/.openclaw` chmod 700; `openclaw.json` chmod 600

### Remote Access
**Tailscale (recommended):** `{ "gateway": { "bind": "100.x.x.x" } }`
**SSH tunnel:** `ssh -N -L 18789:127.0.0.1:18789 user@<lightsail-ip>`

---

## 13. Use Case Playbooks

For full playbook configs and skill recommendations, read [references/playbooks.md](references/playbooks.md).

**Available playbooks:** Personal Executive Assistant, Team Knowledge Bot, Automated Daily Report, GitHub Webhook → Agent Action, Multi-Agent Research Pipeline, Internal IT Helpdesk Bot, Voice + Text Unified Assistant, Secure Enterprise Assistant.

---

## 14. Troubleshooting

| Problem | First Steps |
|---|---|
| Skill not loading | `openclaw skills list`, check `requires.bins`/`requires.env`, verify metadata is single-line JSON |
| Channel not responding | `openclaw doctor`, `openclaw pairing list`, check `allowFrom` and `dmPolicy` |
| Gateway won't start | `openclaw gateway --verbose`, check Node version (22.16+ or 24+), check port 18789 |
| Model auth errors | Verify IAM `bedrock:InvokeModel`, region, model access enabled |
| Sandbox failures | `docker info`, check `setupCommand` for required bins |
| Cron not firing | `openclaw cron list`, check `--tz`, check stagger/OR logic |

---

## 15. Key URLs

| Resource | URL |
|---|---|
| Docs home | https://docs.openclaw.ai |
| Getting started | https://docs.openclaw.ai/start/getting-started |
| Configuration reference | https://docs.openclaw.ai/gateway/configuration |
| Security guide | https://docs.openclaw.ai/gateway/security |
| Channels index | https://docs.openclaw.ai/channels |
| Tools overview | https://docs.openclaw.ai/tools |
| Skills docs | https://docs.openclaw.ai/tools/skills |
| Creating skills | https://docs.openclaw.ai/tools/creating-skills |
| ClawHub (registry) | https://clawhub.ai |
| Scheduled tasks | https://docs.openclaw.ai/automation/cron-jobs |
| AWS Lightsail blueprint | https://docs.aws.amazon.com/lightsail/latest/userguide/amazon-lightsail-quick-start-guide-openclaw.html |
| GitHub | https://github.com/openclaw/openclaw |
| Discord | https://discord.gg/clawd |

---

*Maintained by JJ | Solutions Architecture*
*Last verified: April 23, 2026 against live docs.openclaw.ai and developers.openai.com/codex*