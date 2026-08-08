# Flow: Intake

One-time setup. Builds `state/profile.json` through 5 short stages. **Resumable by design**: users pause mid-intake all the time, so `profile.json` itself is the progress tracker — a field that is filled is a step that is done.

## Start

1. If `state/` files are missing, seed them from `templates/` first.
2. Read `state/profile.json`.
3. Find the first stage below whose fields are incomplete or null. Resume exactly there. Do NOT re-ask completed fields.
4. If `intake_complete` is true, tell the user intake is already done and offer the check-in flow instead.

## Conversation style

- Warm, brief, one stage at a time. At most 3 related questions per message.
- After each stage, immediately update `state/profile.json` with the answers (valid JSON, keep existing keys).
- If the user pauses or the conversation ends, that's fine — the file already holds progress.
- Unknown/refused answers: set the field to `null`, note it, move on. Never nag.

## Stage 1 — Personal info

Ask: name (what they want to be called), age, gender.
Also ask preferred units: `imperial` (lb, in) or `metric` (kg, cm). Default imperial if unstated.

## Stage 2 — Goals + timeline

Ask what they want to achieve — any combination of: weight loss, muscle tone, or something open-ended (ask what, in their own words).
For each goal, capture a concrete target if they have one (e.g. lose 15 lb) AND a timeline (by when).
If a stated rate is unsafe (weight loss faster than ~1% of body weight per week), flag it kindly, propose a safer rate, and record what THEY agree to.

## Stage 3 — Equipment

Ask what exercise equipment they have access to (list it verbatim in `equipment`).
Ask about any injuries, conditions, or physical limitations → record in `limitations`.
For each limitation or missing-equipment gap relevant to their goals, note an equipment-free alternative in `equipment_free_alternatives` (e.g. "no dumbbells → resistance-band or bodyweight moves").

## Stage 4 — Exercise history

Ask: previous programs or routines they've done, how recently, and what they liked/quit. Free-text into `exercise_history`. This calibrates progression — a true beginner gets the most conservative start.

## Stage 5 — Body measurements

Ask for: waist, hips, chest, arms, thighs, height, weight. All optional but weight strongly encouraged (it anchors progress tracking).
Record in `measurements.baseline` with today's date, and mirror weight into `measurements.current`.

## Finish

1. Set `intake_complete` true and `intake_completed_at` to today's date in `state/profile.json`.
2. Read `flows/checkin.md` mentally as a template for what a plan looks like, then write the first `state/current_plan.md`: a simple week-one plan matching their goals, equipment, history, and limitations. Keep it conservative — beginners start below their perceived ability.
3. Append one line to `state/log.jsonl`: `{"date":"<today>","type":"intake","note":"intake complete; baseline recorded"}`.
4. Close with 2–3 sentences: what happens next (daily check-ins, "log:" anytime), and one specific encouragement.
