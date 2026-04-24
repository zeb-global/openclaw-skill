# Core Concepts

## Sessions

Each conversation is a session. Isolated by default. Transcripts stored as JSONL.

**Chat commands:**
| Command | Effect |
|---|---|
| `/status` | Session + model info |
| `/new` | New session |
| `/reset` | Full reset |
| `/compact` | Compact context |
| `/think <level>` | `auto` / `low` / `medium` / `high` |
| `/verbose on\|off` | Verbose output |
| `/trace on\|off` | Trace output |
| `/usage off\|tokens\|full` | Token display |
| `/restart` | Restart gateway |
| `/activation mention\|always` | Group response mode |

## Agents

Multiple agents on one gateway, each with own workspace, model, channel routing, skill allowlist, sandbox policy.

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "model": "bedrock/anthropic.claude-sonnet-4-5"
    },
    "list": [
      { "id": "main" },
      { "id": "docs", "workspace": "~/.openclaw/docs-workspace" }
    ]
  }
}
```

## Models

Format: `provider/model`. Split on first `/`. OpenRouter-style: `openrouter/moonshotai/kimi-k2`.

```json
{ "agents": { "defaults": { "model": "bedrock/anthropic.claude-sonnet-4-5" } } }
```

Use `agents.defaults.models` for primary + fallback list. Provider failover is automatic.
Sub-agent model override: `agents.defaults.subagents.model` (use cheaper model for cost control).

## Queue / Steering

- `steer` — inject inbound messages at next model boundary during current run
- `followup` — hold until turn ends, then start new turn
- `collect` — same as followup but batches messages

## Block Streaming

Off by default. Enable per channel. Controls whether completed blocks send immediately. Tune with `agents.defaults.blockStreamingChunk` (default 800–1200 chars, prefers paragraph breaks).
