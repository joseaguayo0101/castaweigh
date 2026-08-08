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
   └── flows/review.md    ── weekly progress vs goals
        │ read / write
        ▼
   state/  ── LOCAL ONLY, gitignored, never leaves your device
      ├── profile.json      ── who you are, goals, equipment, measurements
      ├── current_plan.md   ── this week's nutrition + exercise plan
      ├── log.jsonl         ── append-only daily log, one line per entry
      └── summary.md        ── rolling long-term memory, updated weekly
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

## Privacy

`state/` is gitignored and stays on your device. The public repo contains only instructions, templates, and the design spec (`castaweigh.json`, a draw.io diagram that is the spec of record). Your health data is never committed and never leaves your control.

## Safety

CastAWeigh is a coaching companion, not a medical professional. It refuses extreme caloric deficits, gives no medical advice, escalates red-flag symptoms, and prioritizes form-first, conservative exercise progression. See the guardrails in `CLAUDE.md`.
