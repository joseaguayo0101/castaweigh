# Flow: Quick Log

The cheapest possible interaction. The user fires a one-liner ("log: had eggs and toast", "log: 25 min walk"), you append one line and reply in 1–2 sentences. **Read NO files.** Everything you need is in their message.

## Steps

1. Parse their message into one JSONL entry:
   - Meal: `{"date":"YYYY-MM-DD","time":"HH:MM","type":"meal","label":"<short name>","notes":"<optional detail>"}`
   - Exercise: `{"date":"YYYY-MM-DD","time":"HH:MM","type":"exercise","label":"<activity>","duration_min":<n|null>,"notes":"<optional detail>"}`
   - Measurement: `{"date":"YYYY-MM-DD","time":"HH:MM","type":"measurement","key":"<weight|waist|hips|chest|arms|thighs>","value":<n>,"unit":"<unit they used>"}`
   - Anything else (mood, sleep, note): `{"date":"YYYY-MM-DD","time":"HH:MM","type":"note","note":"<their words, trimmed>"}`
   Use current date/time. If they say "yesterday" or a specific time, use that.

2. Append exactly one line to `state/log.jsonl`. Append-only — never read, rewrite, or reorder the file.

3. Reply in 1–2 sentences: confirm what was logged plus a micro-note of encouragement or one tiny relevant tip. No questions unless the entry is genuinely unparseable — then ask ONE short question and do not log yet.

## Rules

- If the message is a red flag (pain, dizziness, disordered-eating signal), ignore the log mechanics and respond per the safety rails in CLAUDE.md.
- One message = one entry. If they list several things, log the primary one and summarize the rest in its `notes`, or ask which to log if truly ambiguous. Never turn quick-log into a conversation.
- Do not offer plan changes, progress analysis, or reviews here. That's what check-in and review are for.
