# Signal Inbox

A Claude Code / Cowork plugin that synthesises the last 24 hours of Slack, Gmail, and (optionally) Granola into a single morning brief, written in your voice.

Find what matters before the noise wins.

---

## What you get

A daily brief with five sections:

- **Decisions waiting on you** — with drafted reply sentences in your voice
- **Signals shifting** — political and relationship changes worth noticing
- **Threads going cold** — what you owe people
- **Cross-reference** — meeting commitments not yet actioned (Granola only)
- **One thing to do first** — the highest-leverage action of the day

Plus a footer showing how much noise was filtered out, so you trust what made the cut.

---

## Install

### Claude Code (CLI)

Two commands — add the repo as a marketplace, then install the plugin from it:

```
/plugin marketplace add mrdavemartin/signal-inbox
/plugin install signal-inbox@signal-inbox
```

To install a pinned version:

```
/plugin marketplace add mrdavemartin/signal-inbox@v0.1.0
/plugin install signal-inbox@signal-inbox
```

### Cowork (desktop app)

Cowork doesn't accept GitHub URLs directly, but it does support uploading a custom plugin file:

1. On this repo's GitHub page, click the green **`< > Code`** button → **Download ZIP**.
2. In Cowork, open **Customize** in the left sidebar → **Browse plugins**.
3. Choose the option to **upload a custom plugin** and select the downloaded ZIP.
4. Confirm it appears in your installed plugins list.

To update later, download a fresh ZIP and re-upload — Cowork will replace the existing copy.

---

After install, run `/signal-inbox:doctor` to confirm everything is connected.

---

## First-time setup

1. **Connect Gmail and Slack** in Cowork → Settings → Connectors. (Granola is optional — the brief simply omits the Cross-reference section if it's not connected.)
2. **`/signal-inbox:doctor`** — confirms connectors are reachable and your voice profile is loaded.
3. **`/signal-inbox:setup`** — five short questions that capture your writing voice. Saves to a `voice.md` file. Every brief from then on sounds like you.
4. **`/signal-inbox:brief`** — generate your first brief in chat.
5. **Schedule it** in the Cowork desktop app (see below).

---

## Commands

| Command | What it does |
|---|---|
| `/signal-inbox:help` | Full in-app help with examples. |
| `/signal-inbox:doctor` | Pre-flight check on connectors and voice profile. |
| `/signal-inbox:setup` | One-time voice profile setup. |
| `/signal-inbox:brief` | Generate a brief now. |

### `brief` flags

- `--since 48h` or `--since 2d` — change the lookback window (default 24h, max 7 days)
- `--focus "topic"` — bias the brief toward a specific topic
- `--deliver slack|email|file|inline` — where to send it (default `inline`). Combine with commas: `--deliver slack,file`
- `--channel name` — when delivering to Slack, post to a specific channel (default: your self-DM)
- `--briefs-dir path` — where to save briefs when `--deliver file` is used (default `./briefs/` in the current working directory)

### Examples

```
/signal-inbox:brief
/signal-inbox:brief --since 48h --focus "the Accenture deal"
/signal-inbox:brief --deliver slack,file
/signal-inbox:brief --deliver slack --channel signal-inbox
```

---

## Running it daily

The plugin doesn't schedule itself — Cowork's desktop app handles that natively, with full control over the working folder.

1. Open the Cowork desktop app.
2. Go to **Scheduled Tasks** → **New Task**.
3. **Working folder:** pick where saved briefs should land (e.g. `~/Documents/Signal Inbox/`). Briefs will be written to a `briefs/` subfolder inside it when `--deliver file` is used.
4. **Schedule:** weekdays at 07:00 (or whatever cadence suits).
5. **Prompt:**
   ```
   /signal-inbox:brief --deliver slack,file
   ```
6. Save.

---

## Privacy

Nothing is stored outside your Cowork account. Each brief pulls fresh data from the connected MCPs, sends it to Claude, and renders the result. No analytics, no telemetry, no third-party storage.

Your voice profile lives locally at `~/.claude/plugins/data/signal-inbox/voice.md`.

---

## Customising your voice

`/signal-inbox:setup` is the easiest path. If you'd rather edit the file directly, it lives at:

```
~/.claude/plugins/data/signal-inbox/voice.md
```

The format is open — write whatever instructions help Claude write like you. The shipped default in this repo (`voice.md` at the plugin root) is the fallback when no personal profile exists.

---

## Contributing

Issues and pull requests welcome. Particularly: better noise filters, additional delivery surfaces, voice-profile improvements.

---

## Licence

MIT. See [LICENSE](LICENSE).
