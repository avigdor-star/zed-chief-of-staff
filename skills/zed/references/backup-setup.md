# Backup Setup — AI-Assisted Walkthrough

> Read this when the user has no end-to-end encrypted backup, or when Step 1 of SKILL.md flags the backup check as failing. This document walks the user through choosing one of four free options based on tools they already use.

---

## What backup is (and isn't)

For the snapshot-vs-backup distinction, see `references/vault-safety.md`. This file is the **how** — the AI-assisted walkthrough for setting up off-machine encrypted backup. Two layers of protection: (1) disk encryption (FileVault on Mac, BitLocker on Windows, LUKS on Linux — always on first), and (2) an end-to-end encrypted off-machine copy (rules out plain Google Drive / iCloud / Dropbox / OneDrive — those services *can* read your files; end-to-end encryption means they cannot).

---

## Step 0 — FileVault check (always first)

Ask the user: "Is FileVault turned on? You can check at System Settings → Privacy & Security → FileVault. If it says 'On,' you're good. If it says 'Off,' I'll wait while you turn it on — it takes about 5 minutes and runs in the background."

If FileVault is off → do not proceed until it's on. This is the foundation.

---

## Step 1 — Pick a path

**First, check the user's platform** from `State Dashboard → My Setup → Platform`:
- `notion` → skip everything below. Go straight to **Path E** — Notion has its own backup flow.
- `obsidian`, `logseq`, or `markdown-folder` → continue with the comfort check and path question below (Paths A–D apply to all three identically — the vault is just a folder of files).

### Step 1a — Comfort check (gate Paths B and D behind explicit interest)

Paths B (git-crypt) and D (Syncthing) are more technical. Don't surface them by default — they're noise for users who don't want them, and they bury the simpler paths. Ask first:

> "Two of the backup options are more technical. They give you real things you can't get from the simpler paths:
>
> - **Git-crypt** is the most bulletproof — line-by-line version history with full encryption, the same thing serious developers use for secrets. If you ever need to see exactly what changed between two versions of a briefing six weeks apart, this is the only path that gives you that.
> - **Syncthing** keeps a live encrypted mirror on another machine you own — no cloud, no subscription, fully under your control. If your stance is 'I don't want any cloud company touching my notes, even encrypted,' this is the only path that gives you that.
>
> Do you want me to walk you through these technical options, or stick with the simpler paths (Cryptomator, Proton Drive, or Notion export)?"

- **User says "stick with the simpler paths" / "no" / "skip the technical ones":** offer only Paths A and C. Skip ahead to "Step 1b — Pick a path (simpler options only)."
- **User says "yes" / "walk me through them" / "tell me more":** offer all four paths (A, B, C, D). Continue to "Step 1b — Pick a path (all four options)."

When the user picks B or D, deliver the full step-by-step instructions in that Path's section below — don't dump a wall of shell commands without handholding. Read each step out, wait for the user to confirm before moving to the next, and pause for any error messages.

### Step 1b — Pick a path

**Simpler options only (if user said no in Step 1a):**

> "I'm going to set up off-machine encrypted backup for your Chief of Staff vault. Which of these do you already use or feel comfortable with?
> 1. **iCloud, Dropbox, or Google Drive** (you already pay for or use one of these)
> 2. **None of the above — I want the simplest option**"

- Answer 1 → **Path A: Cryptomator**
- Answer 2 → **Path C: Proton Drive**

**All four options (if user said yes in Step 1a):**

> "I'm going to set up off-machine encrypted backup for your Chief of Staff vault. Which of these do you already use or feel comfortable with?
> 1. **iCloud, Dropbox, or Google Drive** (you already pay for or use one of these)
> 2. **GitHub** (you already have an account)
> 3. **None of the above — I want the simplest option**
> 4. **I want maximum privacy and don't mind a slightly nerdier setup**"

Map their answer:
- Answer 1 → **Path A: Cryptomator** (encrypts the vault, then any cloud syncs the scrambled blobs)
- Answer 2 → **Path B: git + git-crypt** (versioned, encrypted, in a private repo)
- Answer 3 → **Path C: Proton Drive** (free 5 GB, encrypted by default, almost nothing to configure)
- Answer 4 → **Path D: Syncthing** (peer-to-peer to a second device, no cloud at all)

---

## Path A — Cryptomator (recommended for most users)

**What it does in plain language:** Cryptomator creates a special folder on your computer that looks normal to you but looks like scrambled garbage to your cloud sync service. You put the Chief-of-Staff folder inside it. Then iCloud / Dropbox / Drive backs up the scrambled version. If anyone gets into your cloud, they see nothing useful.

**Cost:** Free on desktop. Open-source.

**Walkthrough:**

1. "Download Cryptomator from cryptomator.org. It's free for desktop. Install it like any normal app."
2. "Open Cryptomator. Click 'Add Vault' → 'Create New Vault.'"
3. "Name the vault `chief-of-staff-backup`. Save the vault file inside your iCloud Drive (or Dropbox / Google Drive) folder — wherever your cloud sync lives. This is the encrypted container."
4. "Cryptomator will ask you to create a password. **Write it down somewhere safe — if you lose it, the backup is unrecoverable.** A password manager is the right place."
5. "Cryptomator gives you a recovery key. Save it too. Same warning."
6. "Click 'Unlock.' Cryptomator opens a virtual drive that looks like a normal folder. Drag your `Chief-of-Staff` folder into it. (Or move it — your call. If you move it, update your Cowork folder selection to point to the new location.)"
7. "Done. Your cloud now silently backs up the encrypted version. Anything you save inside the unlocked Cryptomator drive is encrypted before it ever leaves your machine."

**Verify:** Lock and unlock the vault. Confirm the files are still there. Check your cloud — you should see only `.c9r` files (the encrypted blobs).

---

## Path B — git + git-crypt

**What it does in plain language:** Git is a tool that tracks every change you make to files (so you can see history and roll back). git-crypt is an add-on that encrypts the files before they leave your computer. You push the encrypted version to a private GitHub repository. GitHub stores scrambled files; only you can decrypt them.

**Cost:** Free (private repos on GitHub are free; git-crypt is free).

**Best for:** Users comfortable with the command line, who want full version history.

**Walkthrough:**

1. "Do you have a GitHub account? If not, create one at github.com — free."
2. "Install git and git-crypt. On Mac with Homebrew: `brew install git git-crypt`. If you don't have Homebrew, install it from brew.sh first."
3. "Open Terminal. Navigate to the parent of your vault: `cd '~/path/to/your/vault'` (replace with the actual folder name)."
4. "Initialize: `git init` then `git-crypt init`."
5. "Tell git-crypt which files to encrypt. Create a file called `.gitattributes` containing: `Chief-of-Staff/** filter=git-crypt diff=git-crypt`"
6. "Export your key and save it somewhere safe (NOT in the repo): `git-crypt export-key ~/cos-backup-key`. Move that key file to a password manager or a USB drive in a safe."
7. "Create a private repo on GitHub. Name it `chief-of-staff-vault-private`. Do NOT add a README from GitHub's UI."
8. "Connect: `git remote add origin git@github.com:[your-username]/chief-of-staff-vault-private.git`"
9. "Commit and push: `git add . && git commit -m 'initial' && git push -u origin main`"
10. "Verify on GitHub: open the repo in your browser. Open any file in the Chief-of-Staff folder. You should see encrypted gibberish. If you see plain text, something went wrong — stop and tell me."

**Going forward:** the user (or a scheduled task) runs `git add . && git commit -m 'snapshot' && git push` to back up. You can offer to create a daily scheduled task that does this automatically.

---

## Path C — Proton Drive (simplest)

**What it does in plain language:** Proton is a Swiss company that makes encrypted email and file storage. Proton Drive encrypts your files before they leave your machine. You install their app, drop the Chief-of-Staff folder into the Proton Drive sync folder, and you're done. Proton sees only scrambled files.

**Cost:** Free for 5 GB. A text vault is well under that — usually under 100 MB.

**Best for:** Users who want the closest thing to "Dropbox but encrypted" with no configuration.

**Walkthrough:**

1. "Create a free Proton account at proton.me. Pick a strong password. Save it in a password manager."
2. "Download Proton Drive for Mac from proton.me/drive. Install."
3. "Sign in. Proton creates a folder called 'Proton Drive' in your home directory."
4. "Move your `Chief-of-Staff` folder into the Proton Drive folder. (Or copy it first if you're nervous — verify the copy is intact, then delete the original.)"
5. "Update your Cowork folder selection to point to the new location: `~/Proton Drive/My files/Chief-of-Staff/`"
6. "Done. Anything in that folder syncs to Proton, encrypted, automatically."

**Verify:** Log in to drive.proton.me in a browser. Confirm the folder appears. Open a file — Proton decrypts it for you in the browser. Anyone else looking at Proton's servers sees nothing.

---

## Path D — Syncthing (no cloud, peer-to-peer)

**What it does in plain language:** Syncthing copies files directly between two of your own devices (e.g., your Mac and an old laptop, or a NAS, or a phone). No cloud company is ever involved. The transfer is encrypted in transit. Both devices have full copies.

**Cost:** Free. Open-source.

**Best for:** Users who don't trust any cloud at all, and have a second device they own.

**Walkthrough:**

1. "Do you have a second device — old laptop, desktop, NAS, or even a Raspberry Pi — that's powered on at least sometimes?"
2. "Install Syncthing on both devices. Get it from syncthing.net."
3. "On your main Mac, open Syncthing's web UI (it opens automatically at localhost:8384). Add your Chief-of-Staff folder."
4. "On the second device, also open Syncthing. Find the device ID of your Mac in its Syncthing UI."
5. "Pair the two devices by adding each other's device IDs."
6. "Share the Chief-of-Staff folder from the Mac to the second device. Accept the share on the second device. Pick where to store it."
7. "Done. Any change to the Chief-of-Staff folder on either device syncs to the other within seconds."

**Verify:** Edit a file on the Mac. Wait a few seconds. Check the file on the second device. The change should appear.

**Note:** Syncthing is sync, not versioned backup. If the user wants version history, combine with git (Path B), or enable Syncthing's built-in file versioning option.

---

## Path E — Notion (scheduled export + encryption)

**When to use this path:** the user picked `notion` during Step S1.5. Paths A–D above assume a folder of files on the user's machine — Notion doesn't work that way, so Notion users need their own flow.

**What it does in plain language:** Notion encrypts your workspace at rest, but Notion's company can technically read it. To make sure your Chief of Staff content survives a Notion outage, account loss, or worst-case data exposure, we set up two things:

1. A weekly export of your Chief of Staff parent page (and everything under it) into a folder of markdown files on your computer.
2. Encryption of that folder — same tools as the file-based paths above.

That way, even if Notion vanishes tomorrow, you still have an encrypted, off-Notion copy of every briefing, project, and person.

**Cost:** Free.

---

### Part 1 — Set up the weekly export

Pick ONE of these. Option A is more automatic; Option B is more reliable if the Notion connector is flaky.

**Option A — scheduled task in Cowork (more automatic):**

1. "Open a Cowork session. Say: 'Schedule a task called Notion Backup Export to run weekly on Sundays at 9am. Here's the prompt:'"
2. Paste this prompt into the scheduled task:
   ```
   NOTION BACKUP EXPORT — Scheduled Task
   ----------------------------------------
   Run order is fixed: snapshot → export → encrypt. Do NOT reorder.

   1. SNAPSHOT FIRST (hard gate).
      Use the Notion connector to duplicate the State Dashboard page and
      rename the duplicate to:
        state-snapshot-YYYY-MM-DD-HHMM
      Store it under the Archive area.
      If the snapshot fails (API error, permission denied, timeout):
        STOP. Do NOT proceed to export. Post a P1 alert surfacing the failure.
      Rationale: the export that follows takes several minutes and can catch
      the vault mid-edit. The snapshot is the rollback point if the export
      corrupts or partially overwrites anything.

   2. EXPORT.
      Use the Notion connector to fetch the Chief of Staff parent page
      and every descendant (child pages and database records).
      Write each item as a markdown file to:
        ~/Documents/CoS-Notion-Backup/YYYY-MM-DD/
        — Each database row → one .md file, named by the page's Title property.
        — Each regular page body → one .md file, named by the page title.
        — Preserve folder structure matching the Notion parent/child tree.
      If any item fails to fetch: log the error, complete the rest, and post
      a P1 alert with the failed item list. Do NOT proceed to step 3 if more
      than 10% of items failed — stop and surface instead.

   3. ENCRYPT.
      The export folder lives inside the user's chosen encrypted location
      (Cryptomator vault OR Syncthing-shared folder — set during setup).
      Confirm the new dated subfolder is inside that encrypted location.
      If it's outside: move it into the encrypted location, then confirm.

   4. RECORD.
      Update State Dashboard → System Health → Connectors → Backup
      → Last Verified = today's date.
      (This write also takes a snapshot first per the standard gate —
      but the snapshot from step 1 is fresh enough; reuse it rather than
      taking a second one.)
   ```
3. Confirm the task shows up in the user's scheduled task list and will run at the next scheduled time.
4. **Parallel calendar reminder (backup signal).** Also set a recurring weekly calendar reminder: "Notion CoS export — confirm it ran (Sunday 9am)." The scheduled task is primary; this calendar reminder is a human-visible nudge so the user notices if the task silently fails (task didn't trigger, API error not surfaced, etc.). When the reminder fires, spend 30 seconds confirming the latest `~/Documents/CoS-Notion-Backup/YYYY-MM-DD/` folder exists with this week's date.

**Why the order matters (snapshot → export → encrypt).** The Notion export takes several minutes during which the vault is still live — the user could be editing. If the export catches the vault mid-edit and corrupts a record, the snapshot (taken before anything else) is your rollback point. Encrypting before the snapshot would lose that rollback if the encryption step fails. Recording the Last Verified date last means the dashboard only reports success if every previous step actually succeeded.

**Staleness threshold (Option A specific):** Notion exports are weekly, so the standard 30-day staleness window from Step 1a is too loose here — three missed exports would go unflagged. For Option A (and Option B), treat Backup Status as `STALE` if `Last Verified` is more than **10 days** old.

**Option B — manual weekly export (more reliable):**

1. "Set a recurring weekly calendar reminder: 'Notion CoS export — Sunday 9am, 2 minutes.'"
2. "When the reminder fires: open Notion → click your profile icon (top-left) → Settings → go to the Settings tab → scroll down to 'Export content' → click 'Export all workspace content' → choose 'Markdown & CSV' → click Export."
3. "Notion emails you a download link within a few minutes. Download the zip."
4. "Extract the zip into `~/Documents/CoS-Notion-Backup/YYYY-MM-DD/` (create a new folder per export)."
5. **Verify handshake (do this immediately, don't skip).** Open any one Briefing markdown file from the new export folder, confirm it's readable plain text (not gibberish, not empty). Then open the State Dashboard in Notion and update `System Health → Connectors → Backup → Last Verified` to today's date. This mirrors what Option A does automatically — without it, the backup row stays stale and Axiom 1 will keep flagging.

---

### Part 2 — Encrypt the backup folder

You now have a folder of plain markdown on your computer. Encrypt it using one of the tools from the file-based paths:

- **Easiest:** Cryptomator + iCloud/Dropbox/Drive (see **Path A** above). Place the `~/Documents/CoS-Notion-Backup/` folder INSIDE the unlocked Cryptomator vault. Every week's export joins the encrypted pile and syncs scrambled to the cloud.
- **Maximum privacy:** Syncthing (see **Path D** above) to a second device you own. No cloud involved at all.

---

### Verify

After the first export:

1. Open the most recent backup folder. Confirm at least one Briefing and the State Dashboard content are present.
2. Open one `.md` file in any text editor. Confirm it's readable.
3. Confirm the encrypted version exists in your cloud (Cryptomator) or on your second device (Syncthing).
4. Update `State Dashboard → System Health → Connectors → Backup`:

| Connector | Status | Last Verified | Method |
|-----------|--------|---------------|--------|
| Backup | `VERIFIED` | YYYY-MM-DD | Notion-export-encrypted (Cryptomator + iCloud) |

---

### Failure modes specific to Path E

- **Notion API rate limits.** A large workspace export via the connector (Option A) can hit rate limits. If Option A fails repeatedly, fall back to Option B.
- **Notion outage.** During an outage, the scheduled task can't reach Notion and the weekly backup is skipped. Multiple consecutive skips → P1 alert.
- **Forgot to extract the manual export.** Option B requires extracting the zip into the encrypted folder. Skipping that means the latest data isn't actually in the backup. Mitigate with calendar discipline, or switch to Option A.
- **Notion account loss.** If the user loses Notion access (suspension, billing failure), they can still read the encrypted markdown backup — but they can't edit it without migrating to a file-based platform (Obsidian / Logseq / markdown folder). Surface this in the monthly verify: "If Notion disappeared tomorrow, could you still work from the backup? (If no — let's practice the migration once.)"
- **"Export all workspace content" includes non-CoS content.** If the user has other Notion pages outside the Chief of Staff parent, Option B exports those too. This is fine for the backup (more coverage is better), but the user should know the export isn't limited to CoS unless Option A is used.

---

## After any path — record it

Update `State Dashboard → System Health → Connectors → Backup`:

| Connector | Status | Last Verified | Method |
|-----------|--------|---------------|--------|
| Backup | `VERIFIED` | YYYY-MM-DD | Cryptomator + iCloud |

Set the "last verified" date to today. The Chief of Staff will re-verify weekly.

---

## What "verify backup is working" means

A backup that's never been tested is not a backup — it's a hope. "Verify" does NOT mean "the sync indicator is green." It means: **can you actually restore from this backup right now if you had to?** The exact test depends on the path.

### Per-path "verify" procedure

**Path A — Cryptomator:**
1. Quit Cryptomator (lock the vault).
2. Re-open Cryptomator. Unlock the vault using the passphrase from your password manager.
3. Open any one file (e.g., a Briefing) and confirm it's readable plain text.
4. Confirm the cloud sync app (iCloud / Dropbox / Drive) shows the vault as "up to date" — not paused, not "syncing 847 items."

If any step fails: that's a P0 alert — the backup is not actually working.

**Path B — git + git-crypt:**
1. On a new terminal, `cd` to a fresh scratch directory.
2. `git clone [your-repo-url] verify-test` → clone the private repo.
3. `cd verify-test && git-crypt unlock ~/cos-backup-key` (use your saved key file).
4. Open any Briefing `.md` and confirm it's readable plain text (not gibberish).
5. Delete the `verify-test` folder.

If the unlock fails: the key is wrong or missing. That's a P0 alert.

**Path C — Proton Drive:**
1. Open drive.proton.me in a browser (NOT the desktop app — we want to confirm the remote copy is current).
2. Navigate to your Chief-of-Staff folder.
3. Check that the most recent Briefing (dated within the last week) is present.
4. Open that Briefing in the Proton web viewer and confirm it's readable.

If the remote is missing recent files: sync is broken. P0 alert.

**Path D — Syncthing:**
1. Open the Syncthing UI on the SECOND device (not your primary Mac).
2. Confirm the Chief-of-Staff folder shows as "Up to Date" with a sync timestamp within the last 24 hours.
3. Open one recent Briefing directly on the second device and confirm it's readable.

If the second device hasn't synced in 24+ hours: one of the devices is offline. That's not automatically P0, but it is a warning — resolve before this becomes "one copy, not two."

**Path E — Notion export:**
1. Open the most recent dated folder under `~/Documents/CoS-Notion-Backup/YYYY-MM-DD/`.
2. Confirm it contains State Dashboard, at least one Briefing, and the Projects folder.
3. Open one `.md` file in any text editor and confirm it's readable.
4. If Option B (manual export): unzip a prior week's zip to a scratch location and confirm it extracts cleanly — "the zip exists" is not the same as "the zip is valid."

If any step fails: P0 alert — the weekly export is broken.

### Staleness and the monthly trigger

The standard staleness thresholds in SKILL.md Step 1a (Axiom 1) already fire when `Last Verified` is older than the platform's window:

- **Notion** (Path E): `STALE` if `Last Verified` > **10 days** old.
- **All other platforms** (Paths A–D): `STALE` if `Last Verified` > **30 days** old.

In addition, the Chief prompts the user once a month explicitly to verify, even if the dashboard still says `VERIFIED`. This catches silent failures where the scheduled task is "running" but the output is wrong.

**Verbatim monthly prompt:**

> "Monthly backup check. Your current path is [Cryptomator / git-crypt / Proton Drive / Syncthing / Notion export]. To verify it's actually working, I need you to [platform-specific test — see above]. It takes about 2 minutes. Reply 'verified' when done, or tell me what went wrong."

If the user verifies → update `State Dashboard → System Health → Connectors → Backup → Last Verified` to today. If the user can't access the backup → **P0 alert** (3-tier cascade + surface at top of every briefing until resolved).

If the user defers ("not right now") → do not update Last Verified. Axiom 1 keeps surfacing the STALE flag at every briefing until the verify actually happens.

---

## Failure modes to watch for

- **Forgot the password / lost the key:** the backup is unrecoverable. Always store the password and any recovery keys in a password manager (and ideally a printed copy in a safe).
- **Cloud sync paused or out of space:** Cryptomator and Proton both rely on the cloud running. If sync is paused, backup stops. Check the cloud app's sync status during the monthly verify.
- **Repo accidentally made public:** for git-crypt path, double-check the GitHub repo is private. If a git-crypt-protected repo goes public, the encryption still holds, but rotate the key anyway.
- **Second device offline:** for Syncthing, if the second device hasn't been on in a month, you have one copy, not two. Surface this in the monthly check.
