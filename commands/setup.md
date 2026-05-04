---
description: From ConfidenceIn.co Configure your Signal Inbox voice profile. Run this once before your first brief.
argument-hint: ""
---

# Signal Inbox Setup

You are walking the user through a one-time voice profile setup for Signal Inbox. The goal is to produce a `voice.md` file that captures how they write, so every brief sounds like them — not like an AI.

Keep the tone conversational and efficient. No preamble. Ask one question at a time, wait for the answer, then move to the next. Do not explain what you're going to do before doing it — just start with the first question.

---

## Step 1 — Collect answers to five questions

Ask these in order. Wait for each answer before asking the next.

**Q1**
> "Paste 2–3 sentences you've written recently — a Slack message, an email, anything. Don't overthink it."

**Q2**
> "Any words, phrases, or punctuation you never use? List anything that makes you wince when you see it in writing attributed to you."

**Q3**
> "How direct are you? Pick one:
> - A) I lead with the point — trust the reader to handle it
> - B) I soften things a bit — context before conclusion
> - C) I'm diplomatic — relationships come first"

**Q4**
> "British or American English? Anything else about language — industry terms you use, jargon you avoid?"

**Q5**
> "If someone was writing on your behalf for the first time, what's the one thing you'd tell them?"

---

## Step 2 — Synthesise the voice profile

Using their five answers, write a voice profile as a compact set of instructions. Structure it as:

```
## Voice

[2–3 sentence description of their overall style, derived from their writing sample and directness answer]

## Rules
- [rule derived from their answers]
- [rule derived from their answers]
- No em dashes. Ever. Use commas, full stops, or rewrite the sentence.
- [continue for all rules, explicit and inferred]

## Language
[British/American/other, plus any jargon notes]

## In their own words
"[their answer to Q5, lightly tidied if needed but kept in their voice]"
```

The "No em dashes" rule is always included, regardless of whether they mentioned it. Place it naturally among the other rules — do not flag it as special or explain why it's there.

Infer additional rules from the writing sample. If the sample uses short sentences, add "Short sentences. One idea per sentence." If it avoids hedging language, add "No hedging. No 'perhaps', 'it might be worth', 'possibly'." Use your judgement — the goal is a profile that produces output they'd recognise as their own.

---

## Step 3 — Show the profile and confirm

Show the user the synthesised `voice.md` content with this framing:

> "Here's your voice profile. Does this sound right?"

Then show the full profile as a markdown code block so they can read it clearly.

If they say **yes** (or any affirmation): proceed to Step 4.

If they say **no** or flag something off: ask "What's wrong with it?" then revise the specific section and show the updated profile. Repeat until they confirm.

---

## Step 4 — Write the file

Resolve the data directory: use the `CLAUDE_PLUGIN_DATA` environment variable if set and non-empty, otherwise fall back to the plugin root (the directory containing the `commands/` folder).

Write the confirmed profile to `voice.md` inside that directory. Create the directory if it doesn't exist.

Then tell the user:

> "Voice profile saved. Run `/signal-inbox:brief` whenever you're ready."

Nothing else. No summary of what you captured. No congratulations. Just that one line.
