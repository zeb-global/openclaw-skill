# Multi-Agent and Sub-Agent Patterns

## Multi-Agent Config

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "model": "bedrock/anthropic.claude-sonnet-4-5",
      "skills": ["github", "weather"]
    },
    "list": [
      { "id": "main" },
      {
        "id": "docs",
        "workspace": "~/.openclaw/docs-workspace",
        "channels": { "telegram": { "allowFrom": ["@docs-bot"] } }
      },
      {
        "id": "locked",
        "skills": [],
        "tools": { "profile": "messaging" }
      }
    ]
  }
}
```

## Channel Routing to Specific Agents

```json
{
  "agents": {
    "list": [
      { "id": "main", "channels": { "slack": { "workspaces": ["T0TEAMID"] } } },
      { "id": "support", "channels": { "discord": { "servers": ["123456789"] } } }
    ]
  }
}
```

## Sub-Agents

Sub-agents = background agent runs spawned from an existing run. Own session, own context, own token budget. They announce results back to the requester's chat channel when done.

**Slash commands:**
```
/subagents spawn <agentId> <task> [--model <model>] [--thinking <level>]
/subagents list
/subagents log <id|#> [limit] [tools]
/subagents kill <id|all>
/subagents send <id> <message>
/subagents steer <id> <message>
/subagents info <id>
```

**Tool (`sessions_spawn`) params:**
- `task` — required
- `label?` — human label
- `agentId?` — spawn under different agent
- `model?` — override for this run
- `thinking?` — override thinking level
- `runTimeoutSeconds?` — abort after N seconds (default: no timeout)
- `thread?: true` — bind to channel thread
- `mode?: "run" | "session"` — one-shot vs. persistent

**Sub-agent rules:**
- Completion is push-based. Do NOT poll `sessions_history` in a loop waiting for it.
- Each sub-agent has its OWN context + token usage. Use cheaper model: `agents.defaults.subagents.model`.
- Nesting depth configurable. Tool policy tightens at deeper levels by default.
- Session ID format: `agent:<agentId>:subagent:<uuid>`

**Thread-bound sessions:**
```
/focus <subagent-label|session-key>
/unfocus
/session idle <duration|off>
/session max-age <duration|off>
```

## Access Profiles

**Full (trusted user):** `{ "id": "main" }`

**Read-only:**
```json
{
  "id": "reader",
  "sandbox": {
    "mode": "always",
    "docker": {
      "allow": ["read", "sessions_list"],
      "deny": ["bash", "process", "write", "edit", "browser", "cron"]
    }
  }
}
```

**Messaging only:**
```json
{
  "id": "messenger",
  "tools": { "profile": "messaging" },
  "sandbox": { "mode": "always" }
}
```

## ACP Agents (Claude Code, Codex, Gemini CLI)

For harness sessions:
```bash
sessions_spawn with runtime: "acp"
```
See: https://docs.openclaw.ai/tools/acp-agents
