# OpenClaw Agent Skill

The complete reference for deploying, configuring, and operating [OpenClaw](https://openclaw.ai) — the self-hosted personal AI assistant gateway.

Follows the open [AgentSkills](https://agentskills.io) standard. Works with **Kiro, Claude Code, Codex, Cursor**, and any tool that supports SKILL.md.

## Install

### Kiro

1. Open **Agent Steering & Skills** panel → **+** → **Import a skill**
2. Choose **GitHub** → paste `https://github.com/zeb-global/openclaw-skill`

Or manually:
```bash
git clone https://github.com/zeb-global/openclaw-skill.git .kiro/skills/openclaw
```

### Claude Code

```bash
git clone https://github.com/zeb-global/openclaw-skill.git .claude/skills/openclaw
```

### OpenAI Codex

```bash
git clone https://github.com/zeb-global/openclaw-skill.git .agents/skills/openclaw
```

### Cursor

```bash
git clone https://github.com/zeb-global/openclaw-skill.git .cursor/skills/openclaw
```

### OpenClaw (workspace skill)

```bash
git clone https://github.com/zeb-global/openclaw-skill.git ~/.openclaw/workspace/skills/openclaw
```

### Global install (any tool)

Replace `<tool>` with `kiro`, `claude`, `codex`, or `cursor`:
```bash
git clone https://github.com/zeb-global/openclaw-skill.git ~/.<tool>/skills/openclaw
```

## What's Included

- **Architecture** — Gateway, channels, tools, providers
- **Install & Deploy** — Linux, macOS, AWS Lightsail blueprint
- **Configuration** — Workspace files, agents, models, channels (22 total)
- **Skills System** — Writing SKILL.md files, frontmatter, reference patterns
- **Security** — Hardening, sandboxing, DM policies, audit
- **Multi-Agent** — Sub-agents, channel routing, access profiles
- **Automation** — Cron, webhooks, hooks, standing orders
- **Use Case Playbooks** — Executive assistant, knowledge bot, research pipeline, helpdesk, and more

## Structure

```
├── SKILL.md              ← Main skill file (loaded on activation)
├── references/
│   ├── automation.md     ← Cron, webhooks, hooks
│   ├── channels.md       ← All 22 channel configs
│   ├── core-concepts.md  ← Sessions, agents, models
│   ├── multi-agent.md    ← Sub-agents, access profiles
│   ├── playbooks.md      ← 8 use case playbooks
│   ├── security.md       ← Hardening, sandbox, auth
│   ├── skills-system.md  ← Allowlists, token cost, config
│   ├── tools.md          ← Built-in tools, groups, profiles
│   └── workspace-details.md ← SOUL.md, git backup
└── README.md
```

## How It Works

The agent loads only the skill name and description at startup. When your request matches OpenClaw keywords, it loads the full `SKILL.md`. Reference files stay unloaded until the skill body directs the agent to read them — progressive disclosure, minimal token burn.
