# Flow: Briefing (Mode A)

**What this flow does:** Runs a briefing — section-by-section delivery, tailored questions, queued actions, housekeeping at the end.

**When to load it:** User says "briefing," "what's going on," "catch me up," "morning update," "end of day," "what should I focus on," "what's urgent," "did I miss anything" — or a scheduled briefing run fires.

**Prerequisites:** Bootstrap complete. Universal Gates G1-G8 from SKILL.md apply throughout — especially G2 (snapshot), G3 (Filter Rules), G4 (Backup), G5 (Structure).

---

## Step 1 — Source scan

Read these in order, using the platform from `State Dashboard → My Setup → Platform`.

### File-based platforms (`obsidian`, `logseq`, `markdown-folder`)

1. **Index:** `Chief-of-Staff/_index.md` — the vault contract. If a later action doesn't fit the index, the index is right and the action is wrong.
2. **State Dashboard:** `Chief-of-Staff/State Dashboard.md` — current state. Includes the new v2 sections (Surfaced Items, Filter Rules with provenance).
3. **Live Feed:** `Chief-of-Staff/Live Feed.md` — what's happened since last session. **Read the ALERTS section first.**
4. **Tasks (active):** scan `Chief-of-Staff/Tasks/` for files with `status: to-do`, `in-progress`, or `blocked`. Note overdue (due < today). Feeds the TASKS section. Excludes `not-deployed`.
5. **Tasks (awaiting deploy)** (added v2): scan `Chief-of-Staff/Tasks/` for files with `status: not-deployed`. Feeds the AWAITING DEPLOY subsection. Skip if zero.
6. **Reminders today** (added v2): scan `Chief-of-Staff/Reminders/` for files where `status: active AND surface_on <= today AND (snoozed_until is empty OR snoozed_until <= today)`. Feeds REMINDERS TODAY section.
7. **Messaging channel:** recent messages from the team feed channel (Slack `#cos-feed` or equivalent), if connected.
8. **Latest briefing:** most recent file in `Chief-of-Staff/Briefings/`.
9. **Supporting Documents tied to active projects** (added v2): scan `Chief-of-Staff/Supporting-Documents/` for entries linked to projects active in the briefing. List title + Type + Tags.
10. **Legacy DBs (read-only, tagged `[legacy]`)** (added v2): scan `Chief-of-Staff/Research/`, `Chief-of-Staff/Drafts/`, `Chief-of-Staff/Action-Plans/`, `Chief-of-Staff/Reference/`. Cap at 10 entries each per `references/signal-filters.md` § Per-source read budget. Surface entries with `[legacy]` annotation.
11. **Life context (optional):** any life-snapshot or personal-notes file elsewhere in the vault. Personal Journal entries are NOT scanned during briefings (v3) — they're personal and don't need to surface in operational briefings. Recall via `flows/recall.md` if needed.

### Notion (`notion`)

1. **State Dashboard page** — current state (Platform field, Active Projects view, Tasks Due view, High-Priority People view, Watch List, Open Decisions, **Surfaced Items**, **Filter Rules**). Equivalent to `_index.md` + `State Dashboard.md` combined; schema in `references/notion-conventions.md`.
2. **Live Feed page** — **read the Urgent Alerts callout at the top first.**
3. **Tasks (active):** query the Tasks database for Status ∈ {`to-do`, `in-progress`, `blocked`}. Note overdue. Excludes `not-deployed`.
4. **Tasks (awaiting deploy)** (added v2): query for Status = `not-deployed`. Feeds AWAITING DEPLOY.
5. **Reminders today** (added v2): query Reminders database where `Status = active AND surface_on <= today AND (snoozed_until is empty OR snoozed_until <= today)`.
6. **Messaging channel:** if connected.
7. **Latest briefing:** query Briefings database, sort by Date descending, limit 1.
8. **Supporting Documents tied to active projects** (added v2): query Supporting Documents where `Related Projects` includes any active project. List Name + Type + Tags.
9. **Legacy DBs (read-only, tagged `[legacy]`)** (added v2): query Research / Drafts / Action Plans / Reference. Cap at 10 each per signal-filters § Per-source read budget. Surface with `[legacy]` annotation.
10. **Life context (optional):** any life-snapshot page in the broader Notion workspace. Personal Journal entries are NOT scanned during briefings (v3) — they're personal and don't need to surface in operational briefings. Recall via `flows/recall.md` if needed.

> In Notion, if something doesn't fit the schema in `notion-conventions.md`, the schema is right and the action is wrong.

**Monday rule.** On Mondays (or after any gap of 2+ days since last session), scan email and messaging for the whole gap period. "Since last session" = `last_updated` (file-based) or `Last Session At` (Notion). If missing, default to last 72 hours and note the fallback in the briefing.

**Once-per-session legacy nudge** (added v2): if any `[legacy]`-tagged entries surfaced this brief AND `migration_v2_choice` ∈ {`leave-in-place`, `walking`} AND the v2-migration prompt is NOT firing this session: offer once — "I surfaced [N] legacy entries today. Want me to migrate any of them now?" (G9 conflict-detection: at most one legacy-themed prompt per session.)

Apply signal filters per `references/signal-filters.md`.

---

## Step 2 — Filter Rules + Surfaced Items hard gate (G3)

Before delivering ANY item, run it through the filter pipeline in `references/signal-filters.md`:

1. Compute the candidate item's **canonical identifier** (per signal-filters § Canonical Identifiers).
2. Check **Filter Rules** — match by sender / domain / company / topic → drop silently.
3. Check **Surfaced Items state** (added v2) — if the canonical identifier matches an entry in `State Dashboard → Surfaced Items` with state ∈ {`suppress-forever`, `suppress-this-thread`, `handled`} → drop silently.
4. Items with state `active` continue through the pipeline.

A filter rule or Surfaced Items state added after an item was first surfaced still kills it on the next pass. Applies to every section: Top 3, Messages, Carryover, Recommended Actions, Risks/Blockers, Watch List Check, **Reminders Today** (added v2), **Awaiting Deploy** (added v2).

---

## Step 3 — Deliver section by section (strictly linear)

One section at a time. After each section, ask the section's tailored question.

- **Actionable response from user** ("draft a reply to X", "mark Y done", "schedule it") → "Queued for after the brief." Continue.
- **"No" / "nothing" / similar** → ask: **"Can we move on to the next section?"** Only advance on yes.
- **Quick clarifying question** → answer briefly, then re-ask the section's tailored question.
- **"Yes, but not now"** → queue and note "deferred — you can review after the brief."

Never skip sections, never jump ahead on user request.

If a section has no content, announce it ("Messages: nothing") and still ask the move-on question.

### Section order and tailored questions

1. **Alerts** — system-level only: backup (Axiom 1), structure drift (Axiom 2), connector failures, version-constant mismatch (Bootstrap step 0). NOT personal watch-list items.
   - If backup is flagged: pull free slots from the calendar and propose a specific time. ("You're free tomorrow 2–3 PM. Want me to book a backup setup block then?") If yes, create the calendar event.
   - Q: "Want me to handle any of these?"

2. **Reminders Today** (added v2) — Reminders where `Status = active AND surface_on ≤ today AND (snoozed_until is null OR snoozed_until ≤ today)`. Sort per `references/signal-filters.md` § Reminders sort. Hide section if empty.
   - Q: "Want me to handle any of these, snooze, or dismiss?"

3. **Carryover** — unfinished items from the last briefing.
   - Q: "Still relevant, or drop them?"

4. **Schedule** — today's and tomorrow's calendar. Meetings, conflicts, tight windows.
   - Q: "Anything to add or move?"

5. **Top 3** — three most important items for today. Category rank: deadline > people waiting > revenue > strategic > other. Within category, rank Tasks by composite score per `references/signal-filters.md` § Within-category ranking. Tasks with `Time Sensitivity = 5` always surface here regardless of Priority (hard override).
   - Q: "Want me to kick any of these off?"

6. **Tasks** — tasks with Status = to-do, in-progress, or blocked. Excludes `not-deployed` (added v2 — surfaced separately). Overdue first, then due-today, then blocked. Within each bucket, rank by composite score (Priority + Time Sensitivity) per signal-filters.
   - Q: "Any status changes?"

6a. **Awaiting Deploy** (added v2) — Tasks with Status = `not-deployed`, grouped by Project. If 5+ items, render as count + collapsed list ("Awaiting Deploy: 7 items (tap to expand)"). User asks for full list or picks by name. Hide subsection entirely if zero items.
    - Q: "Anything ready to ship today?"

7. **Messages** — emails, DMs, items needing reply or decision. Sender, subject, one-line summary.
   - Q: "Want me to draft a reply to any of these?"

8. **Missed Message Check** — review `State Dashboard → Missed Messages`. Confirm past corrections still honored. Surface patterns flagged multiple times (candidate for permanent filter rule).
   - Q: "Anything I'm still filtering that you want surfaced?"

9. **Watch List** — items from `State Dashboard → Watch List` needing attention today.
   - Q: "Any updates on these?"

10. **Risks / Blockers** — things that could become problems if ignored (deadlines drifting, budget caps nearing, someone waiting too long).
    - Q: "Anything you want to act on?"

11. **Recommended Actions** — specific next steps. "Reply to [person] about [topic]" not "consider following up."
    - Q: "Want me to queue any of these?"

12. **File Cabinet Check** — soft, optional nudge surfacing records without a full parent chain (tasks without a project, projects without a department, departments without a domain). Render ONLY if there is at least one unnested record. Never frame as a violation. Always offer — don't demand. If user wants to home some, hand off to `flows/file-cabinet.md`.
    - Q: "Want to home any of these now, or leave them for later?"

13. **Rollout Nudge** — one soft sentence per the Pacer pattern (`references/rollout-reminder.md`). Render ONLY when ALL are true: `next_eligible ≤ today`, no Axiom 1 or 2 flag this session, nudge hasn't fired this session. If the render gate fails, skip silently (don't even announce as empty).
    - Q (only if rendered): "Want me to start on that now?"

The Chief loads `references/rollout-reminder.md` and uses Section A's decision logic to decide whether to render, and the sentence templates to pick the wording. If the user responds, handle per Sections C–L of that reference.

**Low data day:** "Light day — nothing urgent." Don't fill with filler.
**Connector failure:** State it clearly and continue with what you have.

---

## Step 4 — Run the queue

After the final section, work through queued actions one at a time. Confirm each before executing (G1). When all queued items are handled or deferred, close out.

---

## Step 5 — Proactive documentation (during scan)

While scanning sources, watch for things that should be documented:

- **Operational updates** (task completed, project status changed, new blocker, deadline passed, payment issue) → update the relevant Task, Project, or State Dashboard record. **Apply automatically during housekeeping AND tell the user what you did and why in the briefing itself.** ("Updated the project record for [project] — marked [task] as complete since the deploy went out yesterday.")
- **Working-doc moments** (a brainstorm, prep notes, a draft, a framework — something tied to a project but not a task) → **never auto-write.** Explain why it seems like a Supporting Document and let the user decide. ("That framework you mentioned for [project] sounds like a `plan` Supporting Doc — want me to capture it?")
- **Judgment calls** (not sure where it goes) → offer at the end of the briefing. Keep to 3 items max: "Spotted a few things that might be worth capturing — [short list with where each would go]. Want me to handle any of these?"

**Personal/reflective content is NOT proactively flagged in-brief (v3).** Personal Journal has its own dedicated end-of-brief Personal Check-in section (Step 6 below). Wins, lessons, feelings, and other reflective content are surfaced ONCE per brief in that single predictable touchpoint — not scattered through the brief.

For all logging routing details see `flows/documentation-routing.md`.

---

## Step 6 — Personal Check-in (added v3)

After the queue and proactive documentation are handled — but BEFORE housekeeping — render the Personal Check-in section. This is the single predictable journaling touchpoint per brief.

### Render gate

Before rendering, check `State Dashboard → My Setup → journal_check_in_frequency`:
- `every-brief` (default) → render this brief.
- `weekly` → render only on the first brief of each calendar week (Monday, or earliest brief of the week if no Monday brief).
- `paused` → skip entirely. Don't render.

### What to render

Use the user's chosen Chief Personality (per `references/personality.md`) to phrase. Default wording (Professional):

> "**Personal Check-in.** Anything on your mind today? Want to journal about something, or just note how you're feeling? (yes / nope / not today)"

Personality variations:
- **Playful:** "End-of-brief vibe check 💚 — anything to journal? (yes / nope / not today)"
- **Dry wit:** "Personal check-in (the part of the brief where I ask if you have feelings). Journal? (yes / nope / not today)"
- **Warm:** "Before we wrap — how's your day landing? Want to capture anything? (yes / nope / not today)"

### Apply the user's response

- **Engages** (any pick that opens journaling — "yes," "draft now," supplies content directly, picks one of the 4 routes) → load `flows/journal.md` and pass control. Reset `journal_check_in_skips` to 0 (snapshot first per G2).
- **Skips** (any "no," "nope," "not today," "skip," "next") → increment `journal_check_in_skips` by 1 (snapshot first per G2). If counter reaches 5, surface the soft-mute prompt per `flows/journal.md` § Step 4.
- **Quick clarifying question** → answer briefly, then re-ask.

### When to skip rendering entirely

In addition to the render gate above, suppress the Personal Check-in this brief if:
- `migration_v3_choice` is missing or `pending` AND `flows/version-migration.md` is firing this session — the migration prompt takes priority. The check-in resumes next brief.
- The brief was triggered by an alert auto-fire (not a user-initiated brief). Auto-briefs are operational and shouldn't append a personal prompt.

---

## Step 7 — Housekeeping (after the briefing)

Run in this order. **Snapshot FIRST is the hard gate (G2).**

1. **Snapshot the State Dashboard.** File-based: copy to `Archive/state-snapshot-YYYY-MM-DD-HHMM.md` (HH:MM in 24-hour, local). Notion: duplicate the page via `notion-duplicate-page`, rename to `state-snapshot-YYYY-MM-DD-HHMM`, then `notion-move-pages` it into the Snapshots subpage. Mechanics in `references/vault-safety.md`. **If the snapshot fails: STOP. Do not update the State Dashboard. Surface the failure and ask how to proceed.**
2. Save the briefing to the Briefings folder/database (per the chosen platform's conventions).
3. Update the State Dashboard with any changes.
4. **Update the session anchor.** File-based: set `last_updated:` to now. Notion: update `Last Session At` to now.
5. Update life context with highlights and decisions (if it exists).
6. Sync important messaging items to the Live Feed.
7. Clear handled items from the Live Feed.
8. Mark unhandled items as CARRIED OVER.
9. Post briefing summary to main messaging channel (if connected) — short, scannable.
