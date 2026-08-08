# CastAWeigh

A weight-loss coaching agent that **is** a git repo. No server, no code, no app to install — the repository itself is the agent's complete brain, and your LLM of choice runs it by reading and writing plain markdown and JSON files.

## The concept: repo-as-agent

```
You (phone or desktop)
   │  open the repo with your LLM agent
   ▼
CLAUDE.md  ── the core brain, always loaded (~65 lines)
   │  routes by trigger phrase
   ├── flows/intake.md    ── first-run setup, staged + resumable
   ├── flows/checkin.md   ── daily anchor conversation
   ├── flows/log.md       ── intra-day quick logging (cheapest interaction)
   ├── flows/review.md    ── weekly progress vs goals
   └── flows/sync.md      ── git sync / Drive backup / restore
        │ read / write
        ▼
   state/  ── lives in your PRIVATE repo (never this public one)
      ├── profile.json      ── who you are, goals, equipment, measurements
      ├── current_plan.md   ── this week's nutrition + exercise plan
      ├── log.jsonl         ── append-only daily log, one line per entry
      ├── health.jsonl      ── iPhone Health data, pushed daily by a Shortcut
      └── summary.md        ── rolling long-term memory, updated weekly
        │ git history = versioned backup (automatic)
        │ optional: dated snapshots
        ▼
   Google Drive/CastAWeigh Backups  ── redundancy, restore anytime
```

Token efficiency is the core design constraint:

- `CLAUDE.md` is the only always-loaded file; flows load only when triggered.
- `state/log.jsonl` is append-only — intra-day logging writes one line **without reading anything**.
- `state/summary.md` is the long-term memory, so full history never replays.
- All instructions are plain markdown + JSON. No tool syntax, no vendor-specific features — any capable LLM (Claude, Cursor, or open-source models via Ollama / LM Studio) runs it identically.

## The two repos

| Repo | Visibility | Contents |
|---|---|---|
| `castaweigh` (this one) | **public** | Instructions, templates, diagram — the "binary" you can share. |
| `castaweigh-state` (yours) | **private** | A copy of the above **plus** your live `state/`. This is the repo your agent actually works in. |

Set up your private repo once:

```bash
git clone https://github.com/joseaguayo0101/castaweigh.git castaweigh-state
cd castaweigh-state
git remote rename origin upstream        # keep the public repo as upstream for instruction updates
mkdir -p state
cp templates/profile.json state/profile.json
cp templates/current_plan.md state/current_plan.md
cp templates/log.jsonl state/log.jsonl
cp templates/health.jsonl state/health.jsonl
cp templates/summary.md state/summary.md
sed -i '' '/^state\/$/d' .gitignore      # state/ is tracked in the private repo
git add -A && git commit -m "Seed private state"
# create castaweigh-state as a PRIVATE repo on GitHub, then:
git remote add origin https://github.com/YOURUSER/castaweigh-state.git
git push -u origin main
```

## Running it on your phone (the easy way)

**Cursor for iOS + Cloud Agents** (any paid Cursor plan):

1. Install **Cursor for iOS** (or open [cursor.com/agents](https://cursor.com/agents) in a browser) and connect GitHub with access to your private `castaweigh-state` repo.
2. Start a cloud agent on that repo. The VM clones it, `CLAUDE.md` anchors the session, and you're chatting with your coach — say **"start intake"** the first time.
3. Sessions are conversational: keep sending follow-ups in the same agent and its workspace (and your state) persists between messages.
4. When a run finishes, Cursor opens a PR with the agent's state changes — **one tap to merge** from the phone, with a push notification when it's ready. The diff view doubles as a review of exactly what the agent wrote about you. (Prefer commits straight to `main` with no PR? Launch via the Cloud Agents API with `workOnCurrentBranch: true, autoCreatePR: false` — savable as a one-tap Shortcut POST.)

No files live on the phone at all — the private repo is the single source of truth, and git history is your backup trail.

**Claude Code** works the same way: cloud agents clone the private repo, coach, and push. On desktop, open `castaweigh-state/` as a project and `CLAUDE.md` auto-loads.

**Open-source models (Ollama, LM Studio):** clone the private repo locally, paste `CLAUDE.md` as the system prompt, feed flow files when the model asks for them.

## Talking to your coach

| You say | What runs |
|---|---|
| "start intake" / "let's begin" | `flows/intake.md` — one-time setup (pauses and resumes freely) |
| "morning check-in" / "let's check in" | `flows/checkin.md` — daily anchor: review since last talk, adjust plan |
| "log: had eggs and toast" | `flows/log.md` — one-line append, 1–2 sentence reply |
| "weekly review" / "how am I doing" | `flows/review.md` — progress vs goals, update summary + plan |
| "sync" / "backup" / "done for today" | `flows/sync.md` — commit + push state (plus optional Drive snapshot) |
| "restore" / "new phone" | `flows/sync.md` — clone the private repo, or rebuild from a Drive snapshot |

## iPhone Health integration

A small Apple Shortcut (build guide in `shortcuts/README.md`) runs every morning on a schedule and pushes yesterday's Health summary — steps, exercise minutes, active calories, resting heart rate, sleep, weight — plus each workout, straight into `state/health.jsonl` in your private repo via the GitHub API. Free, no third-party apps; the token it uses can touch only that one repo.

The check-in flow cross-references it so you never re-report what your phone already saw; the weekly review folds activity and sleep trends into your progress assessment. Since only the Shortcut ever writes that file, its commits can never conflict with coaching sessions.

## Backups and restore

Git does the heavy lifting: every sync is a commit, so your private repo's history is a versioned backup you can roll back or clone anywhere. For redundancy, `flows/sync.md` can also save dated snapshot files to a `CastAWeigh Backups` folder in your own Google Drive (free connector — new file each time, nothing overwritten), and restore from the newest snapshot if you ever lose repo access.

## Privacy

This public repo contains only instructions, templates, and the design spec (`castaweigh.json`, a draw.io diagram that is the spec of record). Your data lives in your **private** `castaweigh-state` repo; the agent refuses to push `state/` anywhere else (enforced in `flows/sync.md`). The Shortcut's GitHub token is scoped to read/write that single repo. Optional Drive backups go only to your own Drive folder when you ask.

## Safety

CastAWeigh is a coaching companion, not a medical professional. It refuses extreme caloric deficits, gives no medical advice, escalates red-flag symptoms, and prioritizes form-first, conservative exercise progression. See the guardrails in `CLAUDE.md`.
