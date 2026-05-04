---
description: Show Signal Inbox help — what each command does, with examples.
argument-hint: ""
---

# Signal Inbox Help

Output the help text below, verbatim, as the chat response. No preamble. No additions. No follow-up questions.

---

```markdown
# Signal Inbox

A morning brief that synthesises the last 24 hours across Slack, Gmail, and (optionally) Granola — written in your voice. Find what matters before the noise wins.

---

## First-time setup

1. **Connect Gmail and Slack** in Cowork → Settings → Connectors. (Granola is optional.)
2. **Run `/signal-inbox:doctor`** to confirm everything is reachable.
3. **Run `/signal-inbox:setup`** once to capture your writing voice.
4. **Run `/signal-inbox:brief`** to generate your first brief.
5. **Schedule it** in Cowork (see "Running it daily" below).

---

## Commands

### `/signal-inbox:doctor`
Pre-flight check. Reports which connectors are connected and whether your voice profile is loaded. Tells you exactly what to fix if anything is missing.

### `/signal-inbox:setup`
One-time voice profile setup. Asks five short questions (writing sample, hard rules, directness, language, ghost-writer instruction) and saves the result to `voice.md`. Every brief from then on is written in your voice.

### `/signal-inbox:brief`
Generate a brief now. Five sections:
- **Decisions waiting on you** — with drafted reply sentences in your voice
- **Signals shifting** — political/relationship changes
- **Threads going cold** — what you owe people
- **Cross-reference** — meeting commitments not yet actioned (Granola only)
- **One thing to do first** — the highest-leverage action of the day

**Flags:**
- `--since 48h` or `--since 2d` — change the lookback window (default 24h, max 7 days)
- `--focus "topic"` — bias the brief toward a specific topic
- `--deliver slack|email|file|inline` — where to send it (default: inline). Combine with commas: `--deliver slack,file`
- `--channel name` — when delivering to Slack, post to a specific channel (default: your self-DM)
- `--briefs-dir path` — where to save briefs when `--deliver file` is used. Defaults to `./briefs/` in your current working directory.

### `/signal-inbox:help`
This screen.

---

## Examples

**Generate a brief now, in chat:**
```
/signal-inbox:brief
```

**Last 48 hours, focused on a specific deal:**
```
/signal-inbox:brief --since 48h --focus "the Accenture deal"
```

**Send today's brief to Slack and save a copy:**
```
/signal-inbox:brief --deliver slack,file
```

**Send to a dedicated Slack channel instead of self-DM:**
```
/signal-inbox:brief --deliver slack --channel signal-inbox
```

---

## Running it daily

The plugin doesn't schedule itself — Cowork's desktop app handles that better, with full control over the working folder and timing. Set it up once:

1. Open the **Cowork desktop app**.
2. Go to **Scheduled Tasks** (sidebar) → **New Task**.
3. **Name:** `Signal Inbox brief`
4. **Working folder:** pick a folder where you want saved briefs to land (e.g. `~/Documents/Signal Inbox/`). The brief will write to a `briefs/` subfolder inside it when `--deliver file` is used.
5. **Schedule:** weekdays at 07:00 (or whatever cadence you prefer).
6. **Prompt:** paste this:
   ```
   /signal-inbox:brief --deliver slack,file
   ```
   Adjust the `--deliver` flags to taste. For Slack-only delivery to a dedicated channel:
   ```
   /signal-inbox:brief --deliver slack --channel signal-inbox
   ```
7. Save. The first run will fire at the next scheduled slot.

To pause or change the schedule, edit or disable the task in the same screen.

---

## Files

- **`voice.md`** — your writing voice. Lives in the plugin data directory (`~/.claude/plugins/data/signal-inbox/`, hidden by default, survives plugin updates). Edit directly or re-run `/signal-inbox:setup` to regenerate.
- **Saved briefs** — when you use `--deliver file`, briefs are written to:
  1. The path you pass via `--briefs-dir`, if given
  2. Otherwise, `./briefs/` in the current working directory (which, for scheduled runs, is the working folder you set on the task in Cowork)

---

## Privacy

Nothing is stored outside your Cowork account. Each brief pulls fresh data, sends it to Claude, and renders the result. No analytics, no telemetry, no third-party storage.
```
