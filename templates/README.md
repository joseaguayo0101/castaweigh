# Templates

Blank starting versions of every file in `state/`. These are committed to the repo; your real data is not.

## Setup

Copy these into `state/` before first run (the intake flow can also do this for you):

```bash
mkdir -p state
cp templates/profile.json state/profile.json
cp templates/current_plan.md state/current_plan.md
cp templates/log.jsonl state/log.jsonl
cp templates/health.jsonl state/health.jsonl
cp templates/summary.md state/summary.md
```

## Files

| Template | Becomes | Purpose |
|---|---|---|
| `profile.json` | `state/profile.json` | Empty schema the intake flow fills in. `intake_complete: false` is what triggers intake on first conversation. |
| `current_plan.md` | `state/current_plan.md` | Skeleton the agent fills after intake and adjusts weekly. |
| `log.jsonl` | `state/log.jsonl` | Two example lines showing the entry shapes. Safe to delete the examples once you're logging for real. |
| `health.jsonl` | `state/health.jsonl` | Example lines showing the Health Shortcut's output format. Written only by the shortcut (see `shortcuts/`), read by check-in and review. |
| `summary.md` | `state/summary.md` | Empty rolling-memory skeleton the weekly review maintains. |
