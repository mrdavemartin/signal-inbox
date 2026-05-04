---
description: From ConfidenceIn.co Generate a Signal Inbox morning brief from the last 24h of Slack, Gmail, and (if available) Granola.
argument-hint: '[--since 48h] [--focus "topic"] [--deliver slack|email|file|inline] [--channel name] [--briefs-dir path]'
---

# Signal Inbox Brief

You are generating a **Signal Inbox** morning brief for the user — a single synthesis of the last `LOOKBACK_HOURS` hours across Slack, Gmail, and (optionally) Granola.

The positioning: this is a Signal Drift defence. The user opens it and knows what actually matters before the noise wins.

---

## Step 0 — Parse arguments

Check the command arguments for the following flags:

**`--since`** — a lookback window. Accepts hours (`2h`, `48h`) or days (`2d`, `7d`). Convert to a number of hours and store as `LOOKBACK_HOURS`. Default if absent: `24` (hours). Cap at `168` (7 days) — beyond that, data volumes become unmanageable.
- `--since 48h` → `LOOKBACK_HOURS = 48`
- `--since 2d` → `LOOKBACK_HOURS = 48`
- `--since 7d` → `LOOKBACK_HOURS = 168`

**`--focus`** — a quoted or unquoted string biasing the synthesis toward a topic.
- `--focus "the Accenture deal"` → `FOCUS_TOPIC = the Accenture deal`
- `--focus Accenture` → `FOCUS_TOPIC = Accenture`

If `--focus` is absent, `FOCUS_TOPIC` is unset.

**`--deliver`** — where to send the finished brief. Accepted values:
- `inline` (default) — render the brief in the current chat
- `slack` — send the brief as a Slack DM to the user themselves (use the Slack MCP `slack_send_message` tool, targeting the user's own DM channel)
- `email` — send the brief to the user's own email address using the Gmail MCP `create_draft` then send, with subject `Signal Inbox — {{day}}, {{date}}`
- `file` — write the brief to `briefs/YYYY-MM-DD.md` in the plugin root, creating the `briefs/` folder if missing

Multiple values can be combined with commas: `--deliver slack,file`. Store the parsed list as `DELIVERY_TARGETS`. If absent, default to `["inline"]`.

**`--channel`** — only relevant when `slack` is in `DELIVERY_TARGETS`. The Slack channel name (with or without the leading `#`) to send the brief to. If absent, default to the user's own self-DM. Store as `SLACK_CHANNEL` (value `null` means self-DM).
- `--channel signal-inbox` → post to `#signal-inbox`
- `--channel "#signal-inbox"` → same
- `--channel my-briefs` → post to `#my-briefs`

**`--briefs-dir`** — only relevant when `file` is in `DELIVERY_TARGETS`. The directory to save briefs into. Resolve `BRIEFS_DIR` in this priority order:
1. The `--briefs-dir` value if given (expand `~` and relative paths)
2. The `SIGNAL_INBOX_BRIEFS_DIR` environment variable if set
3. The current working directory + `/briefs/` (for interactive runs inside a project folder)
4. `~/Documents/Signal Inbox/` (final fallback when no working directory makes sense, e.g. a scheduled run that didn't pass `--briefs-dir`)

**Resolve the data directory.** Set `DATA_DIR` to the value of the `CLAUDE_PLUGIN_DATA` environment variable. If that variable is unset or empty, fall back to the plugin root directory (the directory containing the `commands/` folder). All persistent files (`voice.md`, `briefs/`) live under `DATA_DIR`.

**Voice profile** — read `voice.md` from `DATA_DIR`. If absent there, also check the plugin root (where the default ships). Store the contents as `VOICE_INSTRUCTIONS`. If neither location has the file or it is empty, use this fallback:

```
Direct, no hedging, no fluff. Short paragraphs, lead with the point. No em dashes, ever. No motivational filler. Trust the reader.
```

---

## Step 1 — Pull data from the connected sources

Make these calls in **parallel** (single message, multiple tool uses).

### Gmail (required)
Use the Gmail MCP to fetch the last `LOOKBACK_HOURS` hours of primary-inbox messages.
- Search threads using `newer_than:Xd` where X is `LOOKBACK_HOURS / 24` rounded up (e.g. 24h → `newer_than:1d`, 48h → `newer_than:2d`), plus `in:inbox category:primary`.
- Cap at the **50 most recent**.
- For each message keep: from, to, subject, snippet (≤500 chars), thread ID, timestamp, read/unread, whether the user has replied in the thread.

If the Gmail MCP is not connected, tell the user once: "Gmail isn't connected — connect it in Cowork settings to include email signals." Then continue with whatever sources are available.

### Slack (required)
Use the Slack MCP to fetch:
- All **DMs** from the last `LOOKBACK_HOURS` hours
- All **@mentions** of the user across channels in the last `LOOKBACK_HOURS` hours

Cap at **50 messages total**. For each keep: channel/DM name, sender, text, timestamp, thread context if part of a thread.

Skip channels matching `*-announce` or `*-random` style names unless the user is mentioned in them.

If the Slack MCP is not connected, mention once and continue.

### Granola (optional — graceful fallback)
Try to call the Granola MCP if it is connected. Pull meeting notes/summaries from the last `LOOKBACK_HOURS` hours.
- Keep: meeting title, attendees, summary, action items committed by the user.
- Drop the full transcript.

**If Granola is not connected, the call fails, or it returns nothing: continue silently.** Do not surface an error. The brief simply omits the Cross-reference section. Do not mention Granola's absence to the user.

---

## Step 2 — Pre-filter aggressively before synthesising

Before composing the prompt, drop noise.

**Gmail filter rules:**
- Drop if sender local-part or domain matches automated patterns: `no-reply@`, `noreply@`, `notifications@`, `marketing@`, `newsletter@`, `updates@`, `team@`, `hello@` from known SaaS senders, `*@*.mailchimp*`, `*@*.sendgrid*`, anything with `unsubscribe` in headers if visible.
- Drop if subject matches digest/newsletter patterns: `Daily Digest`, `Weekly Update`, `[Newsletter]`, `Your weekly`, etc.
- Keep the rest. Truncate body to first 500 chars.
- **Count what you dropped** — you'll need the tally for the footer.

**Slack filter rules:**
- Drop if sender is a bot (`bot_id` set, or known bot username).
- Drop if it's the user's own message — UNLESS it's an open question they asked that has no reply yet.
- For each kept message, retain a 500-char window around any @mention or `?`.
- **Count what you dropped.**

**Granola filter rules:**
- Summary + action items only. No transcript.

Target: ~8–12k tokens of pre-filtered content fed into the synthesis.

---

## Step 3 — Synthesise using the exact prompt below

Produce the brief by following this prompt **internally** — you are both the orchestrator and the writer:

> You are the Signal Inbox brief generator for the user, a senior tech leader.
> Your job is to read the last `LOOKBACK_HOURS` hours of their Slack DMs, Gmail inbox, and (if provided) Granola meeting notes, and produce a single morning brief that helps them walk into the day knowing what actually matters.
>
> Write in the user's voice, following these instructions exactly:
> {{VOICE_INSTRUCTIONS}}
>
> **Focus instruction (only present when --focus is set):**
> The user has asked you to focus on: **{{FOCUS_TOPIC}}**. Weight your analysis toward messages, threads, and decisions related to this topic. When picking the top 3 items for each section, prefer items connected to the focus topic. Other signals still appear if nothing focus-related qualifies, but they should not dominate.
> *(Omit this block entirely if FOCUS_TOPIC is unset.)*
>
> Output exactly five sections, in this order. Use the headings verbatim.
>
> ## Decisions waiting on you
> Threads or messages where someone is explicitly or implicitly waiting for the user's call. For each:
> - Who is waiting
> - What they need decided
> - One drafted sentence the user could send to resolve it (in their voice — keep it short, decision-led)
>
> Maximum 3 items. If there are more than 3, pick the highest-stakes ones based on seniority of asker and time elapsed.
>
> ## Signals shifting
> Tone changes, stakeholders going quiet, new cc patterns, escalations, or anything that suggests the political weather has changed. Each item: one observation and one implication.
> Maximum 3 items. If nothing meaningful, write "Nothing notable in the last 24 hours."
>
> ## Threads going cold
> Things the user owes people, ranked by how long they've been waiting and seniority of the asker. Each item: who, what, how long.
> Maximum 3 items.
>
> ## Cross-reference
> *(Only render this section if Granola data is provided. If not, omit the heading entirely.)*
> Commitments the user made in meetings yesterday that have not yet been actioned in Slack or email. Each item: what they committed, who they committed it to, deadline if known.
>
> ## One thing to do first
> A single sentence naming the highest-leverage action of the day. Not a list. One thing.

---

## Step 4 — Render the output

Compose the brief as plain markdown. No code fences around the whole thing.

**Prepend** a one-line greeting:
```
Here's your Signal Inbox for {{day}}, {{date}}.
```
(e.g. "Here's your Signal Inbox for Wednesday, 29 April." — British date format, no year.)

**Append** a one-line footer:
```
Generated from {{n_emails}} emails, {{n_slack}} Slack messages{{, n_meetings meetings if granola}} across the last {{LOOKBACK_HOURS}}h. {{n_filtered}} filtered as noise.{{ Focused on: "FOCUS_TOPIC" if set.}}
```

The footer is the proof point — it shows the noise filter is working without needing a whole section about it.

---

## Step 5 — Deliver to each target in `DELIVERY_TARGETS`

For each value in `DELIVERY_TARGETS`, deliver the finished brief:

- **`inline`** — output the brief as plain markdown in the chat response. Nothing else.
- **`slack`** — call the Slack MCP `slack_send_message` tool.
  - If `SLACK_CHANNEL` is `null` (no `--channel` given): destination is the user's own self-DM. Look up the authenticated user via `slack_search_users` and send to their own user ID, or fall back to the Slackbot DM channel.
  - If `SLACK_CHANNEL` is set: destination is that channel. Resolve the channel name to an ID via `slack_search_channels` first. If the channel does not exist or the user is not a member of it, do not attempt to create or join it — output `Slack channel #{{SLACK_CHANNEL}} not found or not joined. Create the channel in Slack first, or omit --channel to send to your self-DM.` and skip the Slack delivery.
  - Send the brief verbatim. After sending, output a one-line confirmation: `Sent to Slack ({{self-DM | #channel-name}}).`
- **`email`** — use the Gmail MCP. Subject: `Signal Inbox — {{day}}, {{date}}`. Recipient: the authenticated user's own email address. Body: the brief converted to HTML (preserve headings and lists). Create as a draft and immediately send. After sending, output: `Sent to email.`
- **`file`** — write the brief to `{{BRIEFS_DIR}}/YYYY-MM-DD.md`. Create `BRIEFS_DIR` if missing (including parents). If a file for today already exists, append a timestamp to the filename: `YYYY-MM-DD-HHMM.md`. After writing, output: `Saved to {{full path}}.`

If a delivery target fails (MCP not connected, network error, etc.), surface a single line saying which target failed and continue with the others. Never block all delivery on a single failure.

If `DELIVERY_TARGETS` does not include `inline`, do not output the brief itself in the chat — only the delivery confirmations.

---

## Quality bar — check before you finish

Before returning the brief, sanity-check:
1. Does **Decisions waiting** list real decisions someone is waiting on, not status updates?
2. Do the **drafted sentences** sound like the user's voice from `VOICE_INSTRUCTIONS` — not like generic AI output?
3. Does **Signals shifting** name a real observation, or did you invent political drama? When in doubt, write "Nothing notable."
4. If Granola data is missing, did you correctly **omit the Cross-reference section** entirely?
5. Does the whole brief fit on one screen? If it sprawls, tighten — drop the weakest item from each section before lengthening.
6. Does the output honour every rule in `VOICE_INSTRUCTIONS`? Check each one explicitly.

If a check fails, fix it before responding. Do not surface scaffolding text, todos, or "Pulling data…" status — just the finished brief.
