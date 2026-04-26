# Watcher Playbook

This document defines every scheduled agent in the Chief of Staff system.

## Two Types of Scheduled Tasks

- **Cloud routines** run on Anthropic's servers (claude.ai/code/scheduled). Work 24/7, even
  when the computer is off. Can access cloud connectors (email, calendar, task tracker,
  messaging) but NOT local files.
- **Desktop tasks** run on the computer through Cowork. Can access everything — local files
  AND cloud connectors. Only run when the computer is awake and Cowork is open.

---

## Write boundaries (strict section ownership)

Every agent in the system has ONE write target. No agent writes to another's target. Any write outside these boundaries is a bug — surface it, don't let it slide.

| Agent | Writes to | Does NOT write to |
|-------|-----------|-------------------|
| **Chief of Staff** (interactive session) | State Dashboard (snapshot FIRST — hard gate), Briefings folder/database, Research/Drafts/Action-Plans folders/databases as tasks are dispatched | Live Feed `## Feed` section (that's watcher territory), Weekly Trends section of State Dashboard (that's Weekly Review territory outside the standard dashboard update) |
| **Watchers** (email / calendar / task tracker / health check / end of day) | Live Feed → `## Feed` section only (append-only) | State Dashboard (ever), Briefings (ever), any other file/database |
| **Weekly Review** | State Dashboard → `Weekly Trends` section, Briefings folder/database (one weekly review record per week) | Live Feed, Projects, People, Research, any watcher territory |
| **Monthly Cleanup** | Archive folder/database, State Dashboard (prune only — never add new content), Briefings (one monthly cleanup record) | Live Feed, active project/person records |

**Snapshot-first rule.** Any write to the State Dashboard (Chief, Weekly Review, Monthly Cleanup) takes a snapshot BEFORE the write. Hard gate — if snapshot fails, the write does not happen.

**Append-only rule.** Watchers only append to Live Feed → `## Feed`. They never modify existing entries and never touch any other section.

If two agents ever share a write target, that's the bug — they must be refactored so one owns it and the other reads-only.

## Setup Instructions

**Desktop task:** Open a Cowork session and say:
"Schedule a task called [Name] to run [frequency]. Here's the prompt: [paste prompt]"

**Cloud routine:** Go to claude.ai/code/scheduled. Create routine, set schedule, paste prompt,
select connectors.

---

## Phase 0 — Manual Mode (Week 1)

Before any automation, just use the system by hand. Open Cowork, say "briefing," and let the
Chief of Staff check your email, calendar, and tasks manually. Do this for a week. This
teaches the system your rhythms and lets you confirm your tool connections work before adding
any automatic checks.

---

## Validation Checklist (before any watchers)

Run these in a Cowork session to confirm your connectors work. After each test, note the
result. If a test fails, check your Cowork connector settings for that tool.

> **Also reused by the Check-in State Pacer.** The Express Lane's `yes` and `walk me through each one` paths re-run the matching check below between each install. See `references/rollout-reminder.md` Section E.

1. **Email sending:** "Send a test email to [your email] with subject 'Test Alert'"
   → If this fails: email sending may not be available. Alerts will use messaging DM + Live Feed instead.
2. **Email reading:** "Check my email for unread messages from the last 24 hours"
   → If this fails: check that your email (Gmail/Outlook) is connected in Cowork settings.
3. **Calendar reading:** "What's on my calendar today?"
   → If this fails: check that your calendar is connected in Cowork settings.
4. **Task tracker** (if connected): "Read my tasks from [your task tracker]"
   → If this fails: check that your task tracker is connected. If you don't use one, skip this.
5. **File writing:** "Append a test entry to Chief-of-Staff/Live Feed.md"
   → If this fails: make sure your Chief-of-Staff folder is connected.
6. **Messaging (Phase 2):** "Send a test message to [your main channel]"
   → Only test this when you're ready for Phase 2.

---

## Phase 1 — Desktop Watchers

### Email Watcher (Desktop — File-based)
**Schedule:** Every 2 hours on weekdays (9am, 11am, 1pm, 3pm)
**Use when platform is:** `obsidian`, `logseq`, or `markdown-folder`
**Migrates to cloud in Phase 2**

```
EMAIL WATCHER — Scheduled Agent (Desktop, File-based)
--------------------------------------------------
1. Read the State Dashboard at: Chief-of-Staff/State Dashboard.md
   → Check High-Priority People, Watch List, and Missed Messages sections.

2. Check email for messages received in the last 3 hours.

3. Apply filters:
   REPORT: real person, asks question/action, High-Priority People list,
   mentions deadline/payment/urgent, matches Missed Messages patterns.
   SKIP: newsletters, automated notifications, FYI-only.

4. DEDUP: Check Live Feed for same sender + subject still marked NEW/CARRIED OVER.

5. Append to Live Feed at: Chief-of-Staff/Live Feed.md (under ## Feed section only — never touch other sections, never touch State Dashboard).
   Format:
   ### [TIMESTAMP] — Email
   **From:** [sender]  **Subject:** [subject]
   **Summary:** [one-line]  **Urgency:** [routine/needs-reply/urgent]
   **Status:** NEW

6. ALERT CHECK: High-Priority People email, payment/billing issue, or
   24+ hours in Live Feed still NEW → 3-tier cascade (email → messaging DM → Live Feed ALERTS).

7. Nothing reportable? Do nothing.
--------------------------------------------------
```

### Email Watcher (Desktop — Notion)
**Schedule:** Every 2 hours on weekdays (9am, 11am, 1pm, 3pm)
**Use when platform is:** `notion`
**Migrates to cloud in Phase 2**

```
EMAIL WATCHER — Scheduled Agent (Desktop, Notion)
--------------------------------------------------
1. Use the Notion connector to read the State Dashboard page
   → Check High-Priority People linked view, Watch List, and Missed Messages sections.

2. Check email for messages received in the last 3 hours (via email connector).

3. Apply filters:
   REPORT: real person, asks question/action, High-Priority People, mentions
   deadline/payment/urgent, matches Missed Messages patterns.
   SKIP: newsletters, automated notifications, FYI-only.

4. DEDUP: Read the Live Feed page's ## Feed section and check for same sender +
   subject still marked NEW / CARRIED OVER.

5. Append to the Live Feed page under the ## Feed section only
   (append-only — never modify existing blocks, never touch any other section,
   never touch the State Dashboard).
   Format (Notion callout or toggle block):
   — Email — [TIMESTAMP]
   From: [sender]  Subject: [subject]
   Summary: [one-line]  Urgency: [routine/needs-reply/urgent]
   Status: NEW

6. ALERT CHECK: High-Priority People email, payment/billing issue, or
   24+ hours in Live Feed still NEW → 3-tier cascade (email → messaging DM → Live Feed Urgent Alerts callout).

7. Nothing reportable? Do nothing.
--------------------------------------------------
```

### Calendar Watcher (Desktop — File-based)
**Schedule:** Daily at 7:15am weekdays
**Use when platform is:** `obsidian`, `logseq`, or `markdown-folder`
**Migrates to cloud in Phase 2**

```
CALENDAR WATCHER — Scheduled Agent (Desktop, File-based)
--------------------------------------------------
1. Read today's and tomorrow's calendar events.
2. Append to Chief-of-Staff/Live Feed.md (under ## Feed section only):
   ### [DATE] — Calendar Update
   TODAY'S SCHEDULE: [events with times]
   TOMORROW PREVIEW: [events]
   FLAGS: [conflicts, back-to-back, prep time issues]
   **Status:** NEW
3. Clear day? Still write: "Clear day — no scheduled meetings."
--------------------------------------------------
```

### Calendar Watcher (Desktop — Notion)
**Schedule:** Daily at 7:15am weekdays
**Use when platform is:** `notion`
**Migrates to cloud in Phase 2**

```
CALENDAR WATCHER — Scheduled Agent (Desktop, Notion)
--------------------------------------------------
1. Read today's and tomorrow's calendar events (via calendar connector).
2. Append to the Live Feed page under the ## Feed section only (append-only —
   never touch State Dashboard, never modify existing blocks):
   — Calendar Update — [DATE]
   TODAY'S SCHEDULE: [events with times]
   TOMORROW PREVIEW: [events]
   FLAGS: [conflicts, back-to-back, prep time issues]
   Status: NEW
3. Clear day? Still write: "Clear day — no scheduled meetings."
--------------------------------------------------
```

### Task Tracker Watcher (Desktop — File-based)
**Schedule:** Daily at 7:30am weekdays
**Use when platform is:** `obsidian`, `logseq`, or `markdown-folder`
**Migrates to cloud in Phase 2**
**Skip if no task tracker is connected.**

```
TASK TRACKER WATCHER — Scheduled Agent (Desktop, File-based)
--------------------------------------------------
1. Read Chief-of-Staff/State Dashboard.md for project context.
2. Scan the connected task tracker:
   - Due dates next 48 hours, overdue, urgent/high priority, status changes, new tasks.
3. DEDUP against Chief-of-Staff/Live Feed.md ## Feed section.
4. Append findings to Chief-of-Staff/Live Feed.md (under ## Feed section only —
   never touch State Dashboard, never modify existing entries).
5. Nothing notable? Do nothing.
--------------------------------------------------
```

### Task Tracker Watcher (Desktop — Notion)
**Schedule:** Daily at 7:30am weekdays
**Use when platform is:** `notion`
**Migrates to cloud in Phase 2**
**Skip if no task tracker is connected.**

```
TASK TRACKER WATCHER — Scheduled Agent (Desktop, Notion)
--------------------------------------------------
1. Use the Notion connector to read the State Dashboard page for project context
   (Active Projects linked view, Watch List).
2. Scan the connected task tracker (task tracker connector — Airtable, Notion
   tasks DB, Linear, Asana, etc.):
   - Due dates next 48 hours, overdue, urgent/high priority, status changes, new tasks.
3. DEDUP by reading the Live Feed page's ## Feed section for recent task entries.
4. Append findings to the Live Feed page under the ## Feed section only
   (append-only — never touch State Dashboard, never modify existing entries).
5. Nothing notable? Do nothing.
--------------------------------------------------
```

### Health Check (Desktop — File-based)
**Schedule:** Daily at 7:45am weekdays
**Use when platform is:** `obsidian`, `logseq`, or `markdown-folder`
**Refocused in Phase 2 to check cloud routine health via messaging**

```
HEALTH CHECK — Scheduled Agent (Desktop, File-based)
--------------------------------------------------
1. Read Chief-of-Staff/Live Feed.md. Check for expected watcher entries:
   - This morning: Calendar (7:15am), Task Tracker (7:30am)
   - Yesterday: at least one Email entry, EOD Review entry
2. Read Chief-of-Staff/State Dashboard.md System Health section (read-only).
3. If anything missing → append a health check entry to Live Feed ## Feed section.
4. If all normal → append a brief "all clear" entry to Live Feed ## Feed section.
5. Update System Health section of State Dashboard — note: this is the ONE
   exception where a watcher writes to the State Dashboard, limited strictly to
   the System Health table's "last check" timestamp. Snapshot first (hard gate).
--------------------------------------------------
```

### Health Check (Desktop — Notion)
**Schedule:** Daily at 7:45am weekdays
**Use when platform is:** `notion`
**Refocused in Phase 2 to check cloud routine health via messaging**

```
HEALTH CHECK — Scheduled Agent (Desktop, Notion)
--------------------------------------------------
1. Use the Notion connector to read the Live Feed page's ## Feed section.
   Check for expected watcher entries:
   - This morning: Calendar (7:15am), Task Tracker (7:30am)
   - Yesterday: at least one Email entry, EOD Review entry
2. Read the State Dashboard page's System Health section (read-only).
3. If anything missing → append a health check entry to the Live Feed page's
   ## Feed section only.
4. If all normal → append a brief "all clear" entry to the Live Feed page's
   ## Feed section only.
5. Update the System Health section of the State Dashboard page — note: this is
   the ONE exception where a watcher writes to the State Dashboard, limited
   strictly to the System Health table's "last check" timestamp. Snapshot the
   State Dashboard page first (hard gate).
--------------------------------------------------
```

### End of Day Review (Desktop — stays desktop always — File-based)
**Schedule:** Daily at 5:00pm weekdays
**Use when platform is:** `obsidian`, `logseq`, or `markdown-folder`

```
END OF DAY REVIEW — Scheduled Agent (Desktop, File-based)
--------------------------------------------------
1. Read today's briefing, Chief-of-Staff/State Dashboard.md (read-only),
   Chief-of-Staff/Live Feed.md, and messaging feed.
2. Append EOD entry to Chief-of-Staff/Live Feed.md ## Feed section only
   (append-only — never touch State Dashboard):
   SYSTEM AWARENESS — priorities status (completed/in progress/unknown)
   Note: "I can only see what's in these files."
   LATE ARRIVALS: anything new since morning briefing
   CARRY TO TOMORROW: items for tomorrow's briefing
3. ALERT CHECK: urgent late arrivals → 3-tier cascade.
4. Post EOD summary to main messaging channel (if connected).
5. Update life context file with brief end-of-day note (if it exists).
--------------------------------------------------
```

### End of Day Review (Desktop — stays desktop always — Notion)
**Schedule:** Daily at 5:00pm weekdays
**Use when platform is:** `notion`

```
END OF DAY REVIEW — Scheduled Agent (Desktop, Notion)
--------------------------------------------------
1. Use the Notion connector to read today's briefing record, the State Dashboard
   page (read-only), and the Live Feed page's ## Feed section. Also read the
   messaging feed if connected.
2. Append EOD entry to the Live Feed page's ## Feed section only (append-only —
   never touch the State Dashboard page):
   SYSTEM AWARENESS — priorities status (completed/in progress/unknown)
   Note: "I can only see what's in these Notion pages."
   LATE ARRIVALS: anything new since morning briefing
   CARRY TO TOMORROW: items for tomorrow's briefing
3. ALERT CHECK: urgent late arrivals → 3-tier cascade.
4. Post EOD summary to main messaging channel (if connected).
5. Update life context page with brief end-of-day note (if it exists).
--------------------------------------------------
```

### Weekly Review (Desktop — stays desktop always)
**Schedule:** Friday at 4:00pm

```
WEEKLY REVIEW — Scheduled Agent (Desktop)
--------------------------------------------------
1. Read all briefings from this week and the State Dashboard.
2. Analyze: what moved, what stalled, patterns, risks, accomplishments.
3. Save weekly summary to Briefings/YYYY-MM-DD-weekly-review.md (use template).
4. Snapshot State Dashboard (in-vault rollback copy, per the standard snapshot-first gate), then update project statuses, watch list, system health.
5. Post weekly summary to main messaging channel (if connected).
--------------------------------------------------
```

### Monthly Cleanup (Desktop — stays desktop always)
**Schedule:** First Monday of month at 9:00am

```
MONTHLY CLEANUP — Scheduled Agent (Desktop)
--------------------------------------------------
1. Move briefings/drafts older than 30 days to Archive/.
2. Snapshot State Dashboard (in-vault rollback copy, per the standard snapshot-first gate), then prune completed projects, stale watch list items,
   resolved missed messages, open decisions older than 30 days.
3. Run memory consolidation on life context and recent journal entries (if they exist).
4. Check Live Feed size — if >200 lines, archive HANDLED entries.
5. Save monthly summary to Briefings/YYYY-MM-monthly-cleanup.md.
--------------------------------------------------
```

---

## Phase 2 — Cloud Routines

Set up after Phase 1 runs smoothly for 2+ weeks AND a messaging platform is connected.
When enabling a cloud version, disable the desktop version to avoid duplicates — use the auto-teardown procedure below.

---

### Messaging channel creation (run this BEFORE any cloud routine)

Phase 2 assumes three messaging channels exist: a feed channel (`#cos-feed`), a check-in channel (`#cos-checkins`), and a main channel (user-chosen, e.g. `#general` or a personal DM). Never assume they exist. Run this procedure the first time the user says "set up Phase 2" or "connect messaging":

1. **Ask which workspace.** "Which messaging workspace should I use? (Slack workspace name, or Teams team name.) If you have more than one, pick the one where you want your Chief of Staff to live."
2. **Ask which main channel.** "Which channel (or DM) do you want for briefing summaries and weekly reviews? It should be somewhere you actually read. I can create a dedicated `#cos-main` if you want, or use an existing channel. Your call."
3. **Check existence.** For each of the three targets (`#cos-feed`, `#cos-checkins`, user-picked main):
   - Use the messaging connector to list channels.
   - For each target: exists already? → skip create.
   - Missing? → create it (private channel recommended — ask the user if they want it private or public).
4. **Confirm membership.** For each channel, confirm the user is a member. If not, add them (Slack: invite to channel; Teams: add as member).
5. **Save channel IDs.** Snapshot the State Dashboard (hard gate), then write the channel IDs to `State Dashboard → System Health → Messaging`:

   | Purpose | Channel name | Channel ID |
   |---------|--------------|------------|
   | Feed | `#cos-feed` | `C0123ABCD` |
   | Check-ins | `#cos-checkins` | `C0456EFGH` |
   | Main | `[user pick]` | `C0789IJKL` |

6. **Post a test to each.** Send a one-line "Hello from your Chief of Staff — this is the [feed / check-in / main] channel." Ask the user to confirm each one arrived. If one fails, stop and debug before continuing.

Do not proceed with any cloud routine setup until all three channels exist, the user is a member, and the test posts landed. The cloud routines post blindly to these channel IDs — if they're wrong, messages go to the wrong place or vanish silently.

---

### Phase 1 → Phase 2 auto-teardown (when a watcher moves from desktop to cloud)

When the user installs the cloud version of a watcher that has a desktop twin (Email, Calendar, Task Tracker, Health Check), the desktop version becomes obsolete. Running both causes duplicates in the feed. Auto-teardown removes the desktop version cleanly.

**Run this immediately after the cloud version's first successful test post.**

1. **Snapshot State Dashboard first (hard gate).** Before any of the following writes, take a snapshot. If the snapshot fails, stop and surface.
2. **Confirm with the user, verbatim:**

   > "I'm about to delete the desktop version of [watcher name] since the cloud version is now running. Okay? (yes / no, keep both for now)"

3. **On `yes`:**
   - Identify the desktop scheduled task by its install name (see the "Identifying a scheduled task for rollback" table at the bottom of this file).
   - Delete the desktop scheduled task using the scheduled-tasks tool.
   - In `State Dashboard → Automation Status`, set the desktop row's Status to `Removed` and the cloud row's Status to `Active`. Set `Last Changed` on both rows to today's date.
   - Confirm back in plain English: "Desktop [watcher name] is off. Cloud version is running. Automation Status updated."

4. **On `no, keep both for now`:**
   - Leave both scheduled tasks in place.
   - In `State Dashboard → Automation Status`, add a Note on the desktop row: "Kept running alongside cloud version at user's request ([today's date]). May produce duplicate Live Feed entries — dedup runs in each watcher."
   - Set a reminder in the State Dashboard: `Pending cleanup — check duplicate rate in 2 weeks`.
   - Do NOT delete.

5. **Watchers this applies to:** Email, Calendar, Task Tracker, Health Check. Does NOT apply to End of Day, Weekly Review, Monthly Cleanup (those stay desktop permanently).

---

### Email Watcher (Cloud)
**Schedule:** Every 2 hours weekdays
**Connectors:** Email provider, Messaging platform
**IMPORTANT:** Set ALERT_THRESHOLD below to match the user's preference from their State
Dashboard. Options: EMERGENCIES_ONLY / IMPORTANT / EVERYTHING.

```
EMAIL WATCHER — Cloud Routine
--------------------------------------------------
NOTE: Cloud routine — no local file access. Post to messaging, not Live Feed.
ALERT_THRESHOLD: [EMERGENCIES_ONLY / IMPORTANT / EVERYTHING] ← paste from State Dashboard
1. Check email for last 3 hours.
2. Filter: report real-person emails with questions/actions/urgency. Skip newsletters/bots.
3. DEDUP against recent feed channel messages.
4. Post to feed channel:
   *Email* — [TIMESTAMP]
   *From:* [sender]  *Subject:* [subject]
   *Summary:* [one-line]  *Urgency:* [level]
5. ALERT: payment/billing, repeated sender, urgent language
   → email user + messaging DM.
--------------------------------------------------
```

**Note:** Cloud routines can't read your files (no file access). Add key senders AND your
alert threshold directly to each cloud routine prompt. The desktop morning briefing always
cross-references the full lists from your State Dashboard.

### Calendar Watcher (Cloud)
**Schedule:** Daily 7:15am weekdays
**Connectors:** Calendar provider, Messaging platform

```
CALENDAR WATCHER — Cloud Routine
--------------------------------------------------
Post to feed channel:
*Calendar Update* — [DATE]
Today's schedule, tomorrow preview, flags for conflicts.
--------------------------------------------------
```

### Task Tracker Watcher (Cloud)
**Schedule:** Daily 7:30am weekdays
**Connectors:** Task tracker, Messaging platform
**Skip if no task tracker is connected.**

```
TASK TRACKER WATCHER — Cloud Routine
--------------------------------------------------
Post to feed channel:
*Tasks Update* — [DATE]
Approaching deadlines, overdue, status changes, new tasks.
--------------------------------------------------
```

### Health Check (Cloud)
**Schedule:** Daily 7:45am weekdays
**Connectors:** Messaging, Email, Calendar, Task tracker
**IMPORTANT:** Set ALERT_THRESHOLD below to match user's preference.

```
HEALTH CHECK — Cloud Routine
ALERT_THRESHOLD: [EMERGENCIES_ONLY / IMPORTANT / EVERYTHING] ← paste from State Dashboard
--------------------------------------------------
1. Check feed channel for expected watcher posts.
2. Test each connector (read one item from each).
3. Post status to feed channel. If connector broken → DM to user.
--------------------------------------------------
```

### Weekend Emergency Check (Cloud)
**Schedule:** Saturday 8:00am
**Connectors:** Email, Messaging

```
WEEKEND EMERGENCY CHECK — Cloud Routine
--------------------------------------------------
ALWAYS use EMERGENCIES_ONLY threshold regardless of user setting.
VERY HIGH threshold. Only flag: emergencies, payment failures, system outages,
someone explicitly needing something before Monday.
If urgent → email + messaging DM. If nothing → silence.
--------------------------------------------------
```

### Check-in Monitor (Cloud)
**Schedule:** Every 30 min weekdays 9am-6pm
**Connectors:** Email, Calendar, Task tracker, Messaging

```
CHECK-IN MONITOR — Cloud Routine
--------------------------------------------------
Read check-in channel for new messages from the user.
Can: check email, calendar, task tracker, answer questions.
Cannot: read local files, take actions, dispatch sub-agents.
Reply in thread. Recommend only — never act.
If question needs file context: "Open a Cowork session for that."
--------------------------------------------------
```

---

## Phased Setup Timeline

The system will remind you when it's time to add the next piece. Check the Automation Status
section in the State Dashboard — the Weekly Review updates it every Friday.

### Phase 0 (Week 1)
- Manual briefings only. No automation. Say "briefing" in Cowork and let the system scan your sources live.
- Run the Validation Checklist to confirm your connections work.

### Phase 1 (Weeks 2-5) — Automatic Background Checks
- Week 2: Automatic email scanning (Email Watcher). Say "set up my email watcher" to install.
- Week 3: Automatic calendar + task tracker scans + health check. Say "set up my calendar watcher."
- Week 4: End of day review. Say "set up my EOD review."
- Week 5+: Weekly review + monthly cleanup. Say "set up my weekly review."

### Phase 2 (Weeks 7-11) — 24/7 Cloud Scans
Requires a messaging platform (Slack or Teams).
- Week 7: Connect messaging, test it, create channels (main, feed, check-in).
- Week 8: Briefings start posting summaries to messaging.
- Week 9: Move email scanning to the cloud (runs even when your computer is off). Go to claude.ai/code/scheduled — you'll see a page where you name your routine, pick the schedule, paste the instructions, and check boxes for which tools it can use.
- Week 10: Move calendar + task tracker scans to the cloud.
- Week 11: Add the between-session check-in monitor + weekend emergency scan.

---

## Identifying a scheduled task for rollback

The Check-in State Pacer's rollback flow (Fix 4f in `references/rollout-reminder.md`) requires the Chief to identify the exact scheduled task for a watcher before deleting it. Lookup depends on where the task actually runs.

| Watcher | Runs where | How to find it |
|---------|-----------|----------------|
| Email Watcher (Desktop, Phase 1) | Cowork scheduled task | Cowork scheduled-tasks list → look for the task name used at install (typically "Email Watcher" or similar) |
| Calendar Watcher (Desktop, Phase 1) | Cowork scheduled task | Same — list Cowork scheduled tasks by name |
| Task Tracker Watcher (Desktop, Phase 1) | Cowork scheduled task | Same |
| Health Check (Desktop, Phase 1) | Cowork scheduled task | Same |
| End of Day Review (Desktop — stays desktop) | Cowork scheduled task | Same |
| Weekly Review (Desktop — stays desktop) | Cowork scheduled task | Same |
| Monthly Cleanup (Desktop — stays desktop) | Cowork scheduled task | Same |
| Email Watcher (Cloud, Phase 2) | Anthropic cloud — claude.ai/code/scheduled | User-facing list on claude.ai/code/scheduled. The Chief cannot delete this automatically — it must walk the user to the page and have them remove it, then update `Automation Status` once the user confirms. |
| Calendar Watcher (Cloud, Phase 2) | Anthropic cloud | Same as Email Watcher (Cloud) |
| Task Tracker Watcher (Cloud, Phase 2) | Anthropic cloud | Same |
| Health Check (Cloud, Phase 2) | Anthropic cloud | Same |
| Weekend Emergency Check (Cloud, Phase 2) | Anthropic cloud | Same |
| Check-in Monitor (Cloud, Phase 2) | Anthropic cloud | Same |

**Rule:** if the watcher runs on the desktop (Cowork scheduled task), the Chief deletes it directly. If it runs in the cloud, the Chief guides the user to `claude.ai/code/scheduled` to remove it, then updates `Automation Status` once the user confirms the removal. The verbatim rollback confirmation in Fix 4f fires BEFORE either path begins.
