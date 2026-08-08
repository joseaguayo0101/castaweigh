# CastAWeigh Health Load — iPhone Shortcut

A free, on-device pipeline that feeds your iPhone's Health data to the agent. Once built, it runs **automatically every morning** and appends yesterday's Health summary + workouts to `state/health.jsonl`, which the check-in and review flows read.

Build time: ~15 minutes in the Shortcuts app. No paid apps, no third-party accounts.

> Since your live state sits in the private GitHub repo (see the main README), the recommended delivery writes straight to that repo via GitHub's API — your phone stays stateless and the data is there before your agent session even starts. Two simpler alternates are at the bottom.

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

## One-time setup: GitHub token (for delivery A)

1. On github.com → Settings → Developer settings → **Personal access tokens → Fine-grained tokens** → Generate new token.
2. Repository access: **Only select repositories** → pick your private `castaweigh-state` repo.
3. Permissions → Repository permissions → **Contents: Read and write**. Nothing else.
4. Generate and copy the token. You'll paste it into the shortcut once. It can only read/write files in that one repo — that's the whole blast radius.

## Build the shortcut

In Shortcuts, create **CastAWeigh Health Load** and add these actions in order (the search bar finds each by name).

### 1. Dates

1. **Date** → Current Date
2. **Adjust Date** → subtract 1 day → *yesterday, the day we summarize*
3. **Format Date** → custom `yyyy-MM-dd` → **Set Variable** `DayLabel`
4. **Date** action on the `yyyy-MM-dd` text of yesterday → converts to midnight at start of yesterday → **Set Variable** `DayStart`
5. Same for today → **Set Variable** `DayEnd`

### 2. Daily metrics

For each row: **Find Health Samples** (Type = metric, filter Start Date ≥ `DayStart` and < `DayEnd`), reduce, then **Set Variable**.

| Variable | Health type | Reduce with |
|---|---|---|
| `Steps` | Steps | **Calculate Statistics** → Sum (round whole) |
| `ExerciseMin` | Apple Exercise Time | Statistics → Sum |
| `ActiveKcal` | Active Energy | Statistics → Sum (kilocalories) |
| `RestingHR` | Resting Heart Rate | Sort Start Date desc → Get First Item → value |
| `Weight` | Body Mass | Sort Start Date desc → Get First Item → value (round 1 decimal) |

Sleep (optional, fiddliest): find **Sleep Analysis** samples with Start Date ≥ 6 hours before `DayStart` and < `DayEnd`, keep only values containing "Asleep" (**Repeat** + **If**), sum durations, **Calculate** ÷ 60, round 1 decimal → `SleepHours`. Skip it if you don't want sleep coaching.

### 3. Compose the new lines

1. **Text** action (insert variables by tapping):

```
{"date":"DayLabel","type":"health","steps":Steps,"exercise_min":ExerciseMin,"active_kcal":ActiveKcal,"resting_hr":RestingHR,"sleep_h":SleepHours,"weight":Weight,"weight_unit":"lb"}
```

→ **Set Variable** `SummaryLine`.

2. **Find Workouts** → filter Start Date ≥ `DayStart` and < `DayEnd`.
3. **Repeat with Each**: inside the loop, a **Text** action:

```
{"date":"DayLabel","type":"workout","label":"Repeat Item › Workout Type","duration_min":Repeat Item › Duration (minutes),"active_kcal":Repeat Item › Total Energy Burned}
```

→ **Add to Variable** `WorkoutLines`.
4. After the loop: **Text** combining `SummaryLine` + `WorkoutLines` (one per line) → **Set Variable** `NewLines`.

### 4. Deliver — pick ONE

**A. GitHub API (recommended — writes into the private state repo)**

1. **Get Contents of URL** — Method GET, URL `https://api.github.com/repos/YOURUSER/castaweigh-state/contents/state/health.jsonl`, Headers: `Authorization: Bearer YOUR_TOKEN` and `Accept: application/vnd.github+json`.
2. **Get Dictionary Value** `content` from the result → **Replace Text** `\n` with nothing (GitHub wraps base64) → **Base64 Decode** → Set Variable `Existing`.
3. **Get Dictionary Value** `sha` from the same result → Set Variable `Sha`.
4. **Text**: `Existing`, newline, `NewLines` → **Base64 Encode** → Set Variable `Encoded`.
5. **Get Contents of URL** — Method PUT, same URL, same headers, Request Body = JSON dictionary: `message` = `"health: DayLabel"`, `content` = `Encoded`, `sha` = `Sha`.
6. Optional: **Show Notification** "Health data pushed" so the morning run confirms itself.

**B. Local file append** (only if you run the agent against a phone-local folder instead of the cloud repo)

- **Append to Text File** → your `castaweigh/state/health.jsonl` (iCloud Drive or On My iPhone). Works with Working Copy locations too — commit/push there occasionally.

**C. Clipboard (zero setup, manual)**

- **Copy to Clipboard** `NewLines`. When you next chat with the agent, paste and prefix with `log: health`. Cheapest thing that works.

### 5. Schedule it (hands-free)

1. Shortcuts → **Automation** → **+** → **Time of Day** → before your usual check-in (e.g. 6:45 AM), repeat **Daily**.
2. **Run Immediately** (no confirmation tap) → select **CastAWeigh Health Load**.

First run prompts for Health read permissions — allow each type. Run once manually with the play button and verify a well-formed line landed in `health.jsonl` (on GitHub for A, in Files for B).

## Notes

- Only the shortcut writes `health.jsonl` — the agent never does — so option A's direct-to-main commits can never conflict with agent sessions.
- If a run fails (token expired, offline), the shortcut just skips; nothing is lost that matters — yesterday's line can be regenerated by re-running manually with the dates shifted.
- The agent reads only the tail of this file (today+yesterday at check-in, ~7 days at review), so growth never costs tokens.
