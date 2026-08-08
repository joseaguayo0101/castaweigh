# Flow: Sync (Drive backup + restore)

State stays local; this flow is how it survives a lost phone, a new device, or a fresh session with no `state/`. Backups go to Google Drive as **new dated files** — never edits to existing files, because that's what Drive integrations reliably support.

Drive folder: `CastAWeigh Backups` (create it on first backup if it doesn't exist).

## When to run

- User says "sync", "backup", "save to Drive", or signals they're done for the day.
- After a weekly review closes, if the last backup is >7 days old, offer a backup in one short sentence.
- User says "restore", "new phone", or `state/` is missing AND the user says they have a Drive backup → run Restore instead of seeding from templates.

## Backup

1. Read all four state files: `state/profile.json`, `state/current_plan.md`, `state/summary.md`, and `state/log.jsonl` (this flow is the ONLY one allowed to read `log.jsonl` in full — a backup needs full fidelity).
2. Bundle them into ONE JSON snapshot:

```json
{
  "app": "castaweigh",
  "backed_up_at": "YYYY-MM-DDTHH:MM",
  "files": {
    "profile.json": { "...profile object..." },
    "current_plan.md": "<full text>",
    "summary.md": "<full text>",
    "log.jsonl": "<full text, one JSON object per line>"
  }
}
```

3. Save it to the `CastAWeigh Backups` Drive folder as `castaweigh-backup-YYYY-MM-DD-HHmm.json`, using the platform's Google Drive capability. If the platform can only create Office/text documents, save the same content as `.txt` — the restore step reads either.
4. Confirm in one sentence: what was saved and where. If the folder already holds many backups (>10), mention that old ones can be deleted manually in Drive — you can't delete files yourself.

## Restore

1. Search the `CastAWeigh Backups` Drive folder for the newest `castaweigh-backup-*` file (any extension).
2. Read it, then write each entry in `files` back to its path under `state/` (create `state/` if needed). Overwrite local files — the backup is the source of truth during a restore.
3. If a backup file is corrupt or unreadable, fall back to the next-newest one and say so.
4. Confirm in 1–2 sentences: backup date restored and a one-line profile recap (name, current goal) so the user can sanity-check it's really their data. Then resume normal routing — usually the check-in flow.

## Rules

- Never back up outside the `CastAWeigh Backups` folder, and never commit `state/` to the public repo.
- Never restore on top of an existing local `state/` unless the user explicitly asked — warn first that local data will be overwritten.
- If no Drive access is available in the current runtime, say so plainly and suggest running sync from a chat that has the Google Drive connector enabled.
