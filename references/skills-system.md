# Skills System Details

## Per-Agent Allowlists

```json
{
  "agents": {
    "defaults": { "skills": ["github", "weather"] },
    "list": [
      { "id": "writer" },
      { "id": "docs", "skills": ["docs-search"] },
      { "id": "locked", "skills": [] }
    ]
  }
}
```

Non-empty list = final set (does not merge with defaults). `[]` = no skills.

## Token Cost

```
total_chars = 195 + Σ (97 + len(name) + len(description) + len(location))
```

~24 tokens per skill plus field lengths. Keep descriptions tight. Only load what the agent needs.

## Skills Config

```json
{
  "skills": {
    "entries": {
      "my-skill": {
        "enabled": true,
        "apiKey": "plaintext-or-{source,provider,id}",
        "env": { "MY_API_KEY": "value" },
        "config": { "endpoint": "https://...", "model": "nano" }
      }
    },
    "load": {
      "extraDirs": ["/shared/skills"],
      "watch": true,
      "watchDebounceMs": 250
    }
  }
}
```

`env` injects into host process only if var not already set. Scoped to agent run.

## Skill Workshop Plugin (auto-generate skills from behavior)

```json
{ "plugins": { "entries": { "skill-workshop": { "enabled": true } } } }
```

Writes to `<workspace>/skills` only. Start with `pending` approval mode.

## Skill Config Key

Config key = skill name by default. If skill sets `metadata.openclaw.skillKey`, use that instead.

## Installer Kinds

`brew`, `node`, `uv`, `go`, `download`. Gateway picks best available (prefers brew if `skills.install.preferBrew`, then uv, then node manager, then go, then download).

## Sandbox Caveat on Bins

`requires.bins` checks the HOST. If agent runs sandboxed, the bin must also exist INSIDE the container. Install via `sandbox.docker.setupCommand`.
