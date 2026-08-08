# Flow: Weekly Review

The deep moment: measure the week against goals and timeline, compress learnings into long-term memory, and adjust next week's plan. Runs ~weekly (triggered by the user, or suggested by check-in).

## Steps

1. Read in this order (once each):
   - `state/profile.json`
   - `state/current_plan.md`
   - `state/summary.md`
   - `state/log.jsonl` — ONLY the tail covering the last ~7 days. Never load the whole file.
   - `state/health.jsonl` — ONLY the tail covering the last ~7 days, and only if the file exists. Skip silently if absent.

2. **Assess vs goals.** Compare the week's log, Health data, and latest measurements against each goal in profile.json and its timeline:
   - Weight/measurement trend vs target rate (safe: ~0.5–1% body weight per week). Prefer Health-captured weight over sporadic manual reports — it's denser.
   - Exercise adherence: planned sessions vs done, counting workouts from `health.jsonl` as done.
   - Activity baseline: average daily steps / active energy across the week — flag a sedentary floor even when workouts happen.
   - Sleep trend if present: chronically short sleep (<~6.5h avg) undermines weight loss; one line, no lecture.
   - Nutrition patterns: what actually got eaten vs plan targets.
   - Reassess rate safety: if loss is faster than ~1%/week, recommend eating more; plateaus under ~3 weeks are normal — say so before changing anything.

3. **Re-measure.** If weight already flows in from `health.jsonl`, skip the weight prompt entirely. If it's been ~2–4 weeks since tape measurements were updated, prompt for a fresh set (waist/hips/chest/arms/thighs, whatever they're willing to do). Append each as a `measurement` line to `state/log.jsonl` and update `measurements.current` in profile.json.

4. **Report** (under ~400 words, plain prose or short lists):
   - Where they stand vs each goal and the timeline — ahead, on track, or behind, in concrete numbers.
   - The week's wins (specific, from the log and Health data) and the one thing that would move the needle most next week.
   - Textual trend summary (e.g. "weight: 182 → 180.4 over 7 days; avg 8.1k steps/day; 4 of 5 planned workouts"). No charts in v1.

5. **Update long-term memory.** Rewrite `state/summary.md` as a rolling summary: keep prior history's key trends, add this week's outcomes, adherence, and decisions. Compress ruthlessly — drop detail older than ~4 weeks into one-liners. This file is what lets full history never replay.

6. **Adjust next week's plan.** Rewrite `state/current_plan.md` for the coming week: progression per the safety rails (≤ ~10% load/volume increase), fixes for what didn't work, respect equipment/limitations exactly. Update `goals` or timeline in profile.json only if the user agrees to a change.

7. Append one line to `state/log.jsonl`: `{"date":"<today>","type":"review","note":"<one-line verdict, e.g. on track, -1.6 lb this week>"}`.

8. **Close.** One motivational note tied to the week's most concrete win.

## Rules

- Behind plan ≠ failure. Frame gaps as information, adjust the plan, never shame.
- All safety rails from CLAUDE.md apply — especially rate-of-loss limits and conservative progression.
- If measurements are missing, review what you have; never block the review on them.
