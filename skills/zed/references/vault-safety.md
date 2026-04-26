# Vault Safety — Snapshots and Backups

> **Applies to:** all four platforms (Obsidian / Logseq / markdown-folder / Notion).
>
> **Loaded when:** a snapshot is about to be written, a snapshot fails, the user asks to restore from a snapshot, or the Chief is explaining the backup vs snapshot distinction.

This file is the single source of truth for snapshots and how they relate to backups. The platform-specific conventions files (`file-based-conventions.md`, `notion-conventions.md`) and the backup walkthrough (`backup-setup.md`) point here instead of duplicating the rules.

---

## Snapshot vs. backup — keep them straight

Two different things. Don't confuse them.

- **Snapshot** = a copy inside your own vault or workspace, used for quick rollback if a write goes wrong.
  - File-based: `Archive/state-snapshot-YYYY-MM-DD-HHMM.md`
  - Notion: a duplicate of the State Dashboard page, named `state-snapshot-YYYY-MM-DD-HHMM`, stored under the `Snapshots` subpage
- **Backup** = the whole vault mirrored to an end-to-end encrypted off-machine location (Cryptomator + iCloud, git-crypt, Proton Drive, Syncthing, or a weekly Notion export). This is survival — what saves you if the disk dies, the laptop is stolen, or Notion vanishes.

**Rules of thumb:**
- Snapshots are useful. Backups are survival.
- Never use the word "backup" to mean an in-vault copy.
- Snapshots are written by the Chief automatically before any State Dashboard update. Backups are set up once during S6 and verified periodically (see `backup-setup.md`).

---

## When to snapshot (the snapshot-first hard gate)

Before any write to the State Dashboard, take a snapshot first. If the snapshot fails (write error, permission denied, Notion API error), STOP — do not perform the write. Surface the failure to the user and ask how to proceed.

**Who must snapshot before writing:**
- Chief of Staff (interactive sessions, briefings)
- Weekly Review
- Monthly Cleanup
- Health Check (the one watcher allowed to update System Health timestamps — still snapshots first)

**Watchers other than Health Check do NOT touch the State Dashboard.** They append to Live Feed only. Snapshot rules don't apply to Live Feed appends.

---

## Snapshot mechanics — file-based

Before writing to `State Dashboard.md`:

1. Copy the current `State Dashboard.md` to `Archive/state-snapshot-YYYY-MM-DD-HHMM.md`. Hour+minute (24-hour, local) is required so multiple sessions on the same day don't overwrite each other.
2. If the exact timestamp filename already exists, append `-2`, `-3`, etc.
3. If the copy fails: STOP. Surface the failure. Do NOT proceed to the write.

**Other file-based snapshot rules:**
- If the Live Feed exceeds 200 lines, move all HANDLED entries to `Archive/feed-archive-YYYY-MM-DD.md`.
- Monthly cleanup moves briefings older than 30 days to `Archive/`.

---

## Snapshot mechanics — Notion

Before writing to the State Dashboard page:

1. Use `notion-duplicate-page` on the State Dashboard. The duplicate lands as a sibling of the original (directly under the Chief of Staff parent) — this is expected.
2. Rename the duplicate to `state-snapshot-YYYY-MM-DD-HHMM` (24-hour local time).
3. Use `notion-move-pages` to move the duplicate into the **Snapshots** subpage (created during S4 — see `notion-conventions.md` → "Snapshot destination").
4. If the exact timestamp title already exists, append `-2`, `-3`, etc.
5. The snapshot is NOT complete until both the duplication and the move succeed. If either step fails: STOP. Surface the failure. Do NOT proceed to the write.

**Why two steps:** `notion-duplicate-page` does not let you pick a destination. The default landing spot is "sibling of the original," which leaves snapshots cluttering the Chief of Staff parent. The move puts them in their proper home.

---

## Snapshot restoration — file-based

Use this when the State Dashboard got corrupted, a session wrote the wrong thing, or the user asks to "restore from snapshot."

**Chief of Staff procedure:**

1. List the three most recent files matching `Archive/state-snapshot-*.md`, sorted by filename descending (newest first).
2. Show the user: "I can restore from one of these snapshots: [timestamps]. Which one?" Do NOT auto-pick — always confirm.
3. Once the user picks one: take a **fresh snapshot of the current (possibly bad) State Dashboard** first, named `state-snapshot-YYYY-MM-DD-HHMM-prerestore.md`. This preserves whatever's there in case the restore was a mistake.
4. Copy the chosen snapshot's contents into `State Dashboard.md` (overwriting it).
5. Confirm to the user in plain English: "Restored State Dashboard from `[filename]`. Saved a copy of what was there before as `[prerestore filename]` in case you want it back."

**Manual restoration (if the user wants to do it themselves):**

1. Open the vault's `Archive/` folder.
2. Find the `state-snapshot-YYYY-MM-DD-HHMM.md` file you want to restore from — pick by timestamp.
3. Copy its contents.
4. Paste into `State Dashboard.md`, replacing everything there. (Save a copy of the current file first if you're not sure.)

Snapshots are just regular markdown files — they can also be opened, diff'd, or partially copied if the user only wants to restore one section.

---

## Snapshot restoration — Notion

**Chief of Staff procedure:**

1. Query the Snapshots subpage for sub-pages matching `state-snapshot-*`. List the three most recent (sort by title descending, since titles are timestamped).
2. Show the user: "I can restore from one of these snapshots: [titles]. Which one?" Do NOT auto-pick — always confirm.
3. Take a fresh snapshot of the current State Dashboard FIRST, named `state-snapshot-YYYY-MM-DD-HHMM-prerestore`. This preserves whatever's there in case the restore was a mistake.
4. Copy the chosen snapshot's content blocks into the live State Dashboard page (replacing existing blocks). The Notion API can fetch all blocks from the snapshot page and append them after deleting the live page's existing blocks.
5. Re-set the page-level properties (`Last Session At`, `setup_status`) from the snapshot's properties so they match.
6. Confirm to the user in plain English: "Restored State Dashboard from `[title]`. Saved a copy of what was there before as `[prerestore title]` in case you want it back."

**Manual restoration (if the user wants to do it themselves):**

1. Open the Snapshots subpage in Notion.
2. Find the snapshot you want — pick by timestamp.
3. Copy its content (Cmd+A inside the page body, then Cmd+C).
4. Paste into State Dashboard, replacing existing content. (Duplicate State Dashboard first if you're not sure — same Cmd+D shortcut.)

---

## What snapshots are NOT

- Snapshots are NOT backups. They live in the same vault/workspace. If the disk dies or the Notion account is lost, snapshots die with it. See `backup-setup.md` for off-machine encrypted backup.
- Snapshots are NOT version history. They're a one-step rollback for the most recent State Dashboard write. For full version history of every file, use git-crypt (`backup-setup.md` Path B).
- Snapshots do NOT cover Live Feed, Briefings, Captain's Log, or any database other than State Dashboard. The snapshot-first hard gate applies only to State Dashboard writes.

---

## Failure handling

If the snapshot step fails:

1. STOP. Do not perform the State Dashboard write.
2. Surface the exact error to the user in plain English.
3. Ask how to proceed. Common options:
   - Retry the snapshot
   - Skip the State Dashboard write entirely (keep the change in working memory, surface it again next session)
   - Investigate the cause (full disk, permission issue, Notion API error)
4. Never silently proceed without a snapshot. The whole point of the gate is that an unrecoverable write is worse than a deferred one.
