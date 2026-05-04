---
description: From ConfidenceIn.co Check that Signal Inbox is ready to run — connectors and voice profile.
argument-hint: ""
---

# Signal Inbox Doctor

You are running a pre-flight check for Signal Inbox. The goal: tell the user in one screen what's working, what's missing, and exactly what to do about each gap. No probing data, no fetching content — this is a connectivity and configuration check only.

Do not output progress chatter ("Checking Slack..."). Run all checks in parallel where possible, then output the final report once.

---

## Checks to run

### 1. Gmail connector (required)
Probe by calling a low-cost Gmail MCP tool — e.g. `mcp__07f95f04-e821-4be2-bee2-b3839cd841f3__list_labels` or equivalent. If the call succeeds, Gmail is connected. If the tool is unavailable or the call errors with auth/permission, Gmail is not connected.

### 2. Slack connector (required)
Probe by calling a low-cost Slack MCP tool — e.g. `mcp__9e4484d2-f007-4ea3-b1de-ea38a991d4f8__slack_search_users` for the authenticated user, or `slack_search_channels` with a tiny query. Success → connected. Tool missing or auth error → not connected.

### 3. Granola connector (optional)
Probe by calling `mcp__c7613b61-e114-4e24-843e-067962913e57__list_meetings` or equivalent. Success → connected. Otherwise → not connected (this is fine, it's optional).

### 4. Voice profile
Resolve the data directory: `CLAUDE_PLUGIN_DATA` if set, otherwise plugin root. Look for `voice.md` first in the data directory, then in the plugin root (where the default ships).
- Custom file in the data directory → "Custom voice profile loaded."
- Only the default file in the plugin root → "Using shipped default voice. Run `/signal-inbox:setup` to personalise it."
- Neither location has it → "No voice profile found. Run `/signal-inbox:setup` or check the plugin install."

---

## Output format

Output a single markdown block, exactly this shape. Use ✅ for OK, ⚠️ for optional/warning, ❌ for blocking.

```
# Signal Inbox — Doctor

## Connectors
✅ Gmail — connected
✅ Slack — connected
⚠️ Granola — not connected (optional, brief will skip the Cross-reference section)

## Configuration
✅ Voice profile — custom profile loaded from voice.md

## Status
Ready. Run `/signal-inbox:brief` to generate today's brief.
```

Adjust the icons and messages based on actual results. Mirror the structure exactly.

---

## Status line rules (the line at the bottom)

- If Gmail **and** Slack are both connected → `Ready. Run /signal-inbox:brief to generate today's brief.`
- If either is missing → `Not ready. Connect {{missing connector(s)}} in Cowork settings, then run /signal-inbox:doctor again.`
- If both are connected but voice.md is missing → keep the "Ready" status, but also include a one-line nudge: `Tip: run /signal-inbox:setup to capture your voice — briefs will sound generic until you do.`

---

## Connection instructions for missing connectors

If any connector is reported as not connected, append a short "How to fix" section after the status line, listing only the missing ones:

```
## How to fix
- **Gmail:** open Cowork → Settings → Connectors → Gmail → Connect.
- **Slack:** open Cowork → Settings → Connectors → Slack → Connect.
- **Granola:** open Cowork → Settings → Connectors → Granola → Connect. (optional)
```

Keep it terse. One bullet per missing connector. Do not explain why each one is needed unless the user asks.
