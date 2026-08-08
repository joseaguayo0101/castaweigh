# CastAWeigh Health Load — iPhone Shortcut

A free, on-device pipeline that feeds your iPhone's Health data to the agent. Once built, it runs **automatically every morning** and appends yesterday's Health summary + workouts to `state/health.jsonl`, which the check-in and review flows read.

Build time: ~10 minutes in the Shortcuts app. No accounts, no paid apps, no cloud — data goes straight from HealthKit into your own castaweigh folder.

> Prefer zero fiddling? The paid app **Health Auto Export** can write the same JSON to a folder on a schedule. This Shortcut is the free equivalent and its output format below is what the agent expects either way.

## What it produces

One daily summary line (per calendar day):

```json
{"date":"2026-08-07","type":"health","steps":8432,"exercise_min":34,"active_kcal":412,"resting_hr":58,"sleep_h":7.2,"weight":183.4,"weight_unit":"lb"}
```

Plus one line per workout that day:

```json
{"date":"2026-08-07","type":"workout","label":"Running","duration_min":28,"active_kcal":310}
```

Every metric except `date` and `type` is optional — if a metric has no data, write `null` for it. The agent tolerates missing fields.

## Build the shortcut

In the Shortcuts app, create a new shortcut named **CastAWeigh Health Load** and add these actions in order. (Search bar finds each by name.)

### 1. Dates

1. **Date** → Current Date
2. **Adjust Date** → subtract 1 day → *this is yesterday, the day we summarize*
3. **Format Date** → custom format `yyyy-MM-dd` → **Set Variable** `DayLabel`
4. **Format Date** (same adjusted date) → custom format `yyyy-MM-dd` → **Date** action to convert that text back to a date → **Set Variable** `DayStart` (this is midnight at the start of yesterday)
5. **Date** → Current Date → **Format Date** → custom `yyyy-MM-dd` → **Date** to convert back → **Set Variable** `DayEnd` (midnight at the start of today)

### 2. Daily metrics

For each metric below: **Find Health Samples** with Type = the metric, filtered to Start Date ≥ `DayStart` and Start Date < `DayEnd`, then reduce it, then **Set Variable**.

| Variable | Health type | Reduce with |
|---|---|---|
| `Steps` | Steps | **Calculate Statistics** → Sum (round to whole number) |
| `ExerciseMin` | Apple Exercise Time | Statistics → Sum |
| `ActiveKcal` | Active Energy | Statistics → Sum (unit: kilocalories) |
| `RestingHR` | Resting Heart Rate | Sort by Start Date descending, Get First Item → value |
| `Weight` | Body Mass | Sort by Start Date descending, Get First Item → value (round 1 decimal) |

Sleep (optional, fiddliest — skip it if you don't care about sleep coaching): find **Sleep Analysis** samples where Start Date ≥ 6 hours before `DayStart` and < `DayEnd`, keep only samples whose value contains "Asleep" (use **Filter**/**Repeat** + **If**), sum their durations, convert minutes→hours with **Calculate** ÷ 60, round to 1 decimal → `SleepHours`.

### 3. Compose and append the daily line

1. **Text** action containing exactly this, with variables inserted (tap to insert each):

```
{"date":"DayLabel","type":"health","steps":Steps,"exercise_min":ExerciseMin,"active_kcal":ActiveKcal,"resting_hr":RestingHR,"sleep_h":SleepHours,"weight":Weight,"weight_unit":"lb"}
```

2. **Append to Text File** → navigate to your `castaweigh/state/` folder → file `health.jsonl`. (Works with iCloud Drive or On My iPhone. If your repo lives in Working Copy, pick its location via the file picker — and commit/push in Working Copy occasionally.)

### 4. Workouts

1. **Find Workouts** → filter Start Date ≥ `DayStart` and < `DayEnd`.
2. **Repeat with Each** over the results. Inside the loop, add a **Text** action:

```
{"date":"DayLabel","type":"workout","label":"Repeat Item › Workout Type","duration_min":Repeat Item › Duration (minutes),"active_kcal":Repeat Item › Total Energy Burned}
```

3. Still inside the loop: **Append to Text File** → same `health.jsonl`.

### 5. Schedule it (hands-free)

1. Shortcuts app → **Automation** tab → **+** → **Time of Day**.
2. Pick a time before your usual check-in (e.g. 6:45 AM), repeat **Daily**.
3. Choose **Run Immediately** (no confirmation tap), then select **CastAWeigh Health Load**.

First run prompts for Health read permissions — tap Allow for each type. Run it once manually with the play button and check that `state/health.jsonl` got a well-formed line.

## Notes for the agent (already encoded in the flows)

- The agent never writes `health.jsonl` — this shortcut owns it.
- Check-in reads only today + yesterday's tail; review reads the last ~7 days.
- If the file doesn't exist (shortcut not set up yet), both flows skip it silently.
