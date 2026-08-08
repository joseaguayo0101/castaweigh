# CastAWeigh

A weight-loss coaching agent that **is** a git repo. No server, no code, no app to install — the repository itself is the agent's complete brain, and your LLM of choice runs it by reading and writing plain markdown and JSON files on your device.

## The concept: repo-as-agent

```
You (phone or desktop)
   │  open this folder with your LLM
   ▼
CLAUDE.md  ── the core brain, always loaded (~120 lines)
   │  routes by trigger phrase
   ├── flows/intake.md    ── first-run setup, staged + resumable
   ├── flows/checkin.md   ── daily anchor conversation
   ├── flows/log.md       ── intra-day quick logging (cheapest interaction)
   ├── flows/review.md    ── weekly progress vs goals
   └── flows/sync.md      ── Drive backup at session end + restore on new device
        │ read / write
        ▼
   state/  ── LOCAL ONLY, gitignored
      ├── profile.json      ── who you are, goals, equipment, measurements
      ├── current_plan.md   ── this week's nutrition + exercise plan
      ├── log.jsonl         ── append-only daily log, one line per entry
      ├── health.jsonl      ── iPhone Health data, written daily by a Shortcut
      └── summary.md        ── rolling long-term memory, updated weekly
        │ end of session: dated snapshot
        ▼
   Google Drive/CastAWeigh Backups  ── your own Drive, restore anytime
```

Token efficiency is the core design constraint:

- `CLAUDE.md` is the only always-loaded file; flows load only when triggered.
- `state/log.jsonl` is append-only — intra-day logging writes one line **without reading anything**.
- `state/summary.md` is the long-term memory, so full history never replays.
- All instructions are plain markdown + JSON. No tool syntax, no vendor-specific features — any capable LLM (Claude, or open-source models via Ollama / LM Studio) runs it identically.

## Quick start

1. Clone this repo to your phone or desktop.
2. Copy the templates into your local state (or let the intake flow do it for you):

```bash
mkdir -p state
cp templates/profile.json state/profile.json
cp templates/current_plan.md state/current_plan.md
cp templates/log.jsonl state/log.jsonl
cp templates/health.jsonl state/health.jsonl
cp templates/summary.md state/summary.md
```

3. Open the folder in your LLM (see below) and say: **"start intake"**.

## Running it

**Claude Code (desktop or mobile):** open the `castaweigh/` folder as a project. `CLAUDE.md` auto-loads as the session anchor — just start talking.

**Claude app:** attach the folder contents in a conversation and paste `CLAUDE.md` as the opening context.

**Open-source models (Ollama, LM Studio, etc.):** paste `CLAUDE.md` as the system prompt, then feed flow files when the model asks for them. Keep the `state/` files open in an editor so you can save the model's updates.

## Talking to your coach

| You say | What runs |
|---|---|
| "start intake" / "let's begin" | `flows/intake.md` — one-time setup (pauses and resumes freely) |
| "morning check-in" / "let's check in" | `flows/checkin.md` — daily anchor: review since last talk, adjust plan |
| "log: had eggs and toast" | `flows/log.md` — one-line append, 1–2 sentence reply |
| "weekly review" / "how am I doing" | `flows/review.md` — progress vs goals, update summary + plan |
| "sync" / "backup" / "done for today" | `flows/sync.md` — dated snapshot of `state/` to your Google Drive |
| "restore" / "new phone" | `flows/sync.md` — rebuild `state/` from your newest Drive backup |

## iPhone Health integration

A small Apple Shortcut (see `shortcuts/README.md`) runs every morning on a schedule and appends yesterday's Health summary — steps, exercise minutes, active calories, resting heart rate, sleep, weight — plus each workout, to `state/health.jsonl`. The check-in flow cross-references it so you never have to re-report what your phone already saw, and the weekly review folds activity and sleep trends into your progress assessment. Free, on-device, no third-party accounts.

## Backups and restore

State lives on your device, but you're not chained to it. Say "sync" (or just wrap up your check-in) and the agent saves one dated snapshot file to a `CastAWeigh Backups` folder in your own Google Drive — new file each time, nothing overwritten. On a new phone or fresh session, say "restore" and it rebuilds `state/` from your newest backup. Requires the Google Drive connector in the Claude app (Settings → Connectors), with file creation enabled.

## Privacy

`state/` is gitignored and stays on your device. The public repo contains only instructions, templates, and the design spec (`castaweigh.json`, a draw.io diagram that is the spec of record). Backups go only to *your* Google Drive when you ask. Health data is pulled on-device by the Shortcut and lands in the same gitignored folder. Nothing is ever committed, and nothing leaves your control except the Drive backups you trigger.

## Safety

CastAWeigh is a coaching companion, not a medical professional. It refuses extreme caloric deficits, gives no medical advice, escalates red-flag symptoms, and prioritizes form-first, conservative exercise progression. See the guardrails in `CLAUDE.md`.
