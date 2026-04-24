# Tools Reference

Three layers: Tools (what the model calls) → Skills (when/how to use them) → Plugins (packages all three).

## Built-in Tools

| Tool | What it does |
|---|---|
| `exec` / `process` | Shell commands, background processes |
| `code_execution` | Sandboxed remote Python |
| `browser` | Chromium (navigate, click, screenshot) |
| `web_search` / `x_search` / `web_fetch` | Web search, X search, page fetch |
| `read` / `write` / `edit` | File I/O in workspace |
| `apply_patch` | Multi-hunk file patches |
| `message` | Send to any connected channel |
| `canvas` | Drive node Canvas |
| `nodes` | Discover and target paired devices |
| `cron` | Manage scheduled jobs |
| `gateway` | Inspect/patch/restart/update gateway |
| `image` / `image_generate` | Analyze or generate/edit images |
| `music_generate` | Generate music |
| `video_generate` | Generate video |
| `tts` | Text-to-speech |
| `sessions_*` / `subagents` / `agents_list` | Session management + sub-agent orchestration |
| `session_status` | Status readback + per-session model override |

`gateway` refuses changes to `tools.exec.ask` or `tools.exec.security`. Use `config.patch` for partial changes; `config.apply` only for full-config replacement.

## Tool Allow/Deny

```json
{
  "tools": {
    "allow": ["group:fs", "browser", "web_search"],
    "deny": ["exec"]
  }
}
```

Deny always beats allow.

## Tool Groups

| Group | Tools |
|---|---|
| `group:runtime` | exec, process, code_execution |
| `group:fs` | read, write, edit, apply_patch |
| `group:sessions` | sessions_list, sessions_history, sessions_send, sessions_spawn, sessions_yield, subagents, session_status |
| `group:memory` | memory_search, memory_get |
| `group:web` | web_search, x_search, web_fetch |
| `group:ui` | browser, canvas |
| `group:automation` | cron, gateway |
| `group:messaging` | message |
| `group:nodes` | nodes |
| `group:agents` | agents_list |
| `group:media` | image, image_generate, music_generate, video_generate, tts |
| `group:openclaw` | All built-in tools (excludes plugin tools) |

## Tool Profiles

| Profile | Includes |
|---|---|
| `full` | No restriction (default) |
| `coding` | group:fs, group:runtime, group:web, group:sessions, group:memory, cron, media |
| `messaging` | group:messaging, sessions list/history/send, session_status |
| `minimal` | session_status only |

Per-agent: `agents.list[].tools.profile`. Per-provider: `tools.byProvider.<provider>.profile`.

## Plugin-Provided Tools

- **Lobster** — typed workflow runtime with resumable approvals
- **LLM Task** — JSON-only structured output from model
- **OpenProse** — markdown-first workflow orchestration
- **Diffs** — diff viewer and renderer
- **Memory Wiki** — wiki-style persistent memory
- **Voice Call** — voice call integration
