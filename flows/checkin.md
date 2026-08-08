# Flow: Daily Check-in

The daily anchor conversation. Read the state, catch up since last talk, log it, adjust the plan, send them off motivated.

## Steps

1. Read in this order (once each, then never again this conversation):
   - `state/profile.json`
   - `state/current_plan.md`
   - `state/summary.md`
   - `state/log.jsonl` — ONLY the tail: entries from today and the previous day. Never load the whole file.

2. **Catch up.** Ask what they've eaten and what exercise they've done since the last conversation (check log timestamps to know when that was). Compare against the plan. One or two questions, not an interrogation.

3. **Log it.** For each meal, exercise, or measurement they report, append one line to `state/log.jsonl`:
   - Meal: `{"date":"YYYY-MM-DD","time":"HH:MM","type":"meal","label":"<short name>","notes":"<what/amount, optional>"}`
   - Exercise: `{"date":"YYYY-MM-DD","time":"HH:MM","type":"exercise","label":"<activity>","duration_min":<n|null>,"notes":"<sets/reps/intensity, optional>"}`
   - Weight/measurement if reported: `{"date":"YYYY-MM-DD","time":"HH:MM","type":"measurement","key":"weight","value":<n>,"unit":"<from profile>"}` — and update `measurements.current` in profile.json.

4. **Coach.** Briefly: how today/the plan is going vs their goals. One concrete recommendation for the rest of the day or tomorrow — diet or exercise, whatever matters most right now. Exercise suggestions must respect their equipment, alternatives, and limitations from the profile. Keep form cues for anything new.

5. **Adjust the plan if needed.** Small in-place edits to `state/current_plan.md` only when something changed (missed days, new constraints, food preferences). Don't rewrite the plan daily; the weekly review owns big changes.

6. **Close.** One motivational note tied to something specific they actually did or said. Never generic praise.

## Rules

- Short paragraphs. No headers, no bullet dumps. This is a conversation, not a report.
- If they report pain, red-flag symptoms, or disordered-eating signals, the safety rails in CLAUDE.md take over — stop coaching and respond per those rules.
- If it's been ~7 days since the last review entry in summary.md, suggest running the weekly review.
