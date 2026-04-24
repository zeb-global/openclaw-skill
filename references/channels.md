# Channel Setup

## DM Policy (critical — read before exposing any channel)

Default: `dmPolicy: "pairing"`. Unknown senders receive a code; bot ignores until approved.

```bash
openclaw pairing approve <channel> <code>
openclaw pairing list
openclaw doctor    # surfaces risky DM configs
```

**Options:**
- `pairing` (default) — unknown senders gated by code
- `open` — all DMs processed (requires `allowFrom: ["*"]`; trusted bots only)
- `disabled` — reject all DMs

## Config Patterns

**Telegram:**
```json
{
  "channels": {
    "telegram": {
      "token": "...",
      "dmPolicy": "pairing",
      "allowFrom": ["@yourusername"]
    }
  }
}
```

**Slack:**
```json
{
  "channels": {
    "slack": {
      "appToken": "xapp-...",
      "botToken": "xoxb-...",
      "dmPolicy": "pairing",
      "allowFrom": ["U1234567890"]
    }
  }
}
```

**Discord:**
```json
{
  "channels": {
    "discord": {
      "token": "...",
      "dmPolicy": "pairing",
      "allowFrom": ["123456789012345678"]
    }
  }
}
```

**WhatsApp:**
```json
{ "channels": { "whatsapp": { "allowFrom": ["+1234567890"] } } }
```

**Microsoft Teams:**
```json
{
  "channels": {
    "teams": { "tenantId": "...", "botId": "...", "dmPolicy": "pairing" }
  }
}
```

## Group Chats

Default: responds only when mentioned. Use `/activation always` to respond to all messages. Sandbox non-main sessions: `agents.defaults.sandbox.mode: "non-main"`.

## All 22 Channels

WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage (BlueBubbles), iMessage (legacy), IRC, Microsoft Teams, Matrix, Feishu, LINE, Mattermost, Nextcloud Talk, Nostr, Synology Chat, Tlon, Twitch, Zalo, WeChat, QQ, WebChat.
