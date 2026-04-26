# Flow: Watcher Setup

**What this flow does:** Walks the user through phased rollout of automatic background scans (watchers). One-at-a-time by default; Express Lane only if the user explicitly asks.

**When to load it:** User says "set up my watchers," "install Phase 1," "set up automatic checks," "express lane," "install everything," "do them all," or names a specific watcher to install.

**Prerequisites:** Bootstrap complete. Universal Gates G1-G8 apply. Read `references/watcher-playbook.md` for every watcher's exact prompt, schedule, and write boundary; read `references/rollout-reminder.md` for the Pacer / Express Lane / rollback logic.

---

## Step 1 — Check what's already running

Read `State Dashboard → Automation Status` to see what's active. The phase the user is currently on, what's installed, and what hasn't been touched yet.

---

## Step 2 — Disambiguate: one-at-a-time or Express Lane?

**One-at-a-time (default for "set up my watchers", "install Phase 1"):** walk through each scheduled task in sequence. After each one, confirm it's working, then move to the next. Recommended path for most users.

**Express Lane (only if user explicitly says "install everything", "express lane", "all at once", "do them all now"):** install all remaining watchers in Phase 1 in one pass, per the Express Lane section of `references/rollout-reminder.md`.

**Express Lane warning script (verbatim — speak this before any Express Lane install):**

> "I'm hearing you want to install all remaining pieces now — [list of watchers that would be added, e.g. 'email watcher, calendar watcher, end-of-day review, weekly review']. That skips the gradual baseline: if something breaks later, it'll be harder to figure out which piece caused it because everything got added at once. Want me to proceed anyway? (yes / no / walk me through each one)"

Then route per `references/rollout-reminder.md` Section E (the three branches: yes / no / walk me through each one).

---

## Step 3 — Per-watcher install

For each watcher being installed (one-at-a-time or inside Express Lane):

1. **Pick the right variant** from `references/watcher-playbook.md` — branch on the user's platform (file-based vs Notion) and on whether it's a Phase 1 (desktop) or Phase 2 (cloud) watcher.
2. **Create the scheduled task yourself** using the scheduled-task tool — don't ask the user to copy-paste from a reference doc. Use the exact prompt block from the playbook.
3. **Run the matching validation** from the Validation Checklist at the top of `references/watcher-playbook.md`. Examples: send-test-email check, calendar-read check, file-write check.
4. **Confirm to the user in plain English:** "Installed. Test passed — [specific result, e.g. 'test entry appeared in your Live Feed']."
5. If validation fails: stop the sequence, surface the error, treat as mid-abort if inside Express Lane (per Section F of `references/rollout-reminder.md`).

---

## Step 4 — Update Automation Status

Snapshot the State Dashboard first (G2). Then in `State Dashboard → Automation Status`, update the row for the watcher just installed: Status `Active`, Last Changed today.

If anything went wrong with the snapshot: STOP. Do not update Automation Status. Surface to the user.

---

## Phase 2 prerequisites

If the user is moving to Phase 2 (cloud watchers), messaging channels must exist first. See `references/watcher-playbook.md` → "Messaging channel creation" before any cloud routine setup. Don't proceed with cloud watchers until all three channels (`#cos-feed`, `#cos-checkins`, user-picked main) exist, the user is a member, and the test posts landed.

---

## Phase 1 → Phase 2 auto-teardown

When installing the cloud version of a watcher that has a desktop twin (Email, Calendar, Task Tracker, Health Check), the desktop version becomes obsolete. Run the auto-teardown procedure in `references/watcher-playbook.md` → "Phase 1 → Phase 2 auto-teardown" immediately after the cloud version's first successful test post.

---

## Rollback

If the user wants to remove a watcher: load `references/rollout-reminder.md` → Section H (Rollback). The verbatim "Rollback confirmation" template is mandatory before deleting. Branch on whether the watcher is desktop (DELETE_DESKTOP path — Chief deletes directly) or cloud (GUIDE_CLOUD path — Chief walks the user through manual deletion at claude.ai/code/scheduled).
