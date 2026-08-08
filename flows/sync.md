# Flow: Sync (git-first, Drive as optional backup)

How state survives across sessions, devices, and lost phones. Two layers, both free:

1. **Git (primary)** — state lives in the user's PRIVATE repo (`castaweigh-state`), which contains a copy of this repo's instructions plus `state/`. Every sync is a commit; history is a free, versioned backup trail.
2. **Google Drive (optional redundancy)** — dated snapshot files via the platform's Drive connector, for the day git isn't reachable.

## When to run

- User says "sync", "backup", "save", or signals they're done for the day.
- After a weekly review closes, offer a sync in one short sentence if one hasn't happened.
- "restore" / "new phone" → run Restore below.

## Sync (git)

If this runtime has a shell and network access (Cursor cloud agents, Claude Code, any desktop):

1. `git pull --rebase` — picks up anything written since this session started (e.g. the Health Shortcut appending `health.jsonl` via the GitHub API while you were coaching).
2. Stage the `state/` files (never anything else): `git add state/`
3. Commit: `git commit -m "state: YYYY-MM-DD sync"` — amend nothing, force nothing.
4. `git push`
5. Confirm in one sentence. If pull/push fails (offline, conflict), say so plainly and fall back to the Drive snapshot below if the user wants a backup now.

Runtimes without a shell (plain chat): skip git silently and use the Drive path below if the user wants a backup.

## Drive snapshot (optional, redundancy)

1. Read all four state files: `state/profile.json`, `state/current_plan.md`, `state/summary.md`, and `state/log.jsonl` (this flow is the ONLY one allowed to read `log.jsonl` in full — a backup needs full fidelity). Include `state/health.jsonl` too if it exists.
2. Bundle into ONE JSON snapshot:

```json
{
  "app": "castaweigh",
  "backed_up_at": "YYYY-MM-DDTHH:MM",
  "files": {
    "profile.json": { "...profile object..." },
    "current_plan.md": "<full text>",
    "summary.md": "<full text>",
    "log.jsonl": "<full text, one JSON object per line>",
    "health.jsonl": "<full text, one JSON object per line>"
  }
}
```

3. Save it to the `CastAWeigh Backups` Drive folder (create it on first use) as `castaweigh-backup-YYYY-MM-DD-HHmm.json`, using the platform's Google Drive capability. If the platform can only create Office/text documents, save the same content as `.txt` — restore reads either.
4. Confirm in one sentence. Old snapshots can be cleaned up manually in Drive — you can't delete files yourself.

## Restore

- **Git path (preferred):** the user clones the private `castaweigh-state` repo — state comes with it. Done. One sentence confirming the latest commit date and a one-line profile recap (name, current goal) so they can sanity-check it's their data.
- **Drive path (fallback):** find the newest `castaweigh-backup-*` file in the `CastAWeigh Backups` folder, read it, write each entry in `files` back to its path under `state/` (create `state/` if needed; the backup is the source of truth). If it's corrupt, fall back to the next-newest and say so.
- If the user is mid-setup on a NEW device with neither repo access nor Drive, seed from `templates/` instead and tell them their history will return at the next git pull or Drive restore.

## Rules

- NEVER commit or push `state/` to the public `castaweigh` repo — only to the private `castaweigh-state` repo. If the remote URL is the public repo, stop and warn.
- Never back up outside the `CastAWeigh Backups` Drive folder.
- Never restore on top of an existing local `state/` unless the user explicitly asked — warn first that local data will be overwritten.
- If neither git nor Drive is available in the current runtime, say so plainly and name what would work (Cursor cloud agent, Claude Code, or a chat with the Drive connector).
