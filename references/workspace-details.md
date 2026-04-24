# Workspace Details

## SOUL.md — Personality

`SOUL.md` is where voice lives. Short and sharp beats long and vague.

**Put here:** tone, opinions, brevity rules, humor, bluntness, when to push back.
**Do NOT put here:** changelogs, security policies, life story, corporate speak.

Good rules: "have a take", "skip filler", "call out bad ideas early", "be funny when it fits"
Bad rules: "maintain professionalism at all times", "provide comprehensive assistance"

**Molty rewrite prompt:**
```
Read your SOUL.md. Rewrite it: strong opinions, delete corporate language, ban "Great question"/"Absolutely", mandate brevity, allow humor, allow calling out dumb ideas with charm, allow swearing when it lands. Save as SOUL.md.
```

## Git Backup (recommended — keep private)

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md memory/
git commit -m "init"
# Add private GitHub/GitLab remote. NEVER commit secrets.
```

## Skip Bootstrap Creation

```json
{ "agent": { "skipBootstrap": true } }
```
