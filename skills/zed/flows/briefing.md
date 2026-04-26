# Flow: Briefing (Mode A)

**What this flow does:** Runs a briefing — section-by-section delivery, tailored questions, queued actions, housekeeping at the end.

**When to load it:** User says "briefing," "what's going on," "catch me up," "morning update," "end of day," "what should I focus on," "what's urgent," "did I miss anything" — or a scheduled briefing run fires.

**Prerequisites:** Bootstrap complete. Universal Gates G1-G8 from SKILL.md apply throughout — especially G2 (snapshot), G3 (Filter Rules), G4 (Backup), G5 (Structure).

---

## Step 1 — Source scan

Read these in order, using the platform from `State Dashboard → My Setup → Platform`.

### File-based platforms (`obsidian`, `logseq`, `markdown-folder`)

1. **Index:** `Chief-of-Staff/_index.md` — the vault contract. If a later action doesn't fit the index, the index is right and the action is wrong.
2. **State Dashboard:** `Chief-of-Staff/State Dashboard.md` — current state.
3. **Live Feed:** `Chief-of-Staff/Live Feed.md` — what's happened since last session. **Read the ALERTS section first.**
4. **Tasks:** scan `Chief-of-Staff/Tasks/` for files with `status: to-do`, `in-progress`, or `blocked`. Note overdue (due < today). Feeds the TASKS section.
5. **Messaging channel:** recent messages from the team feed channel (Slack `#cos-feed` or equivalent), if connected.
6. **Latest briefing:** most recent file in `Chief-of-Staff/Briefings/`.
7. **Captain's Log (last 7 days):** list entries in `Chief-of-Staff/Captain's Log/` with dates in the last 7 days. Titles + Type only.
8. **Life context (optional):** any journal, life-snapshot, or personal-notes file elsewhere in the vault.

### Notion (`notion`)

1. **State Dashboard page** — current state (Platform field, Active Projects view, Tasks Due view, High-Priority People view, Watch List, Open Decisions). Equivalent to `_index.md` + `State Dashboard.md` combined; schema in `references/notion-conventions.md`.
2. **Live Feed page** — **read the Urgent Alerts callout at the top first.**
3. **Tasks:** query the Tasks database for Status = `to-do`, `in-progress`, or `blocked`. Note overdue.
4. **Messaging channel:** if connected.
5. **Latest briefing:** query Briefings database, sort by Date descending, limit 1.
6. **Captain's Log (last 7 days):** query the top-level `Captain's Log` database. Titles + Type only.
7. **Life context (optional):** any life-snapshot or journal page in the broader Notion workspace.

> In Notion, if something doesn't fit the schema in `notion-conventions.md`, the schema is right and the action is wrong.

**Monday rule.** On Mondays (or after any gap of 2+ days since last session), scan email and messaging for the whole gap period. "Since last session" = `last_updated` (file-based) or `Last Session At` (Notion). If missing, default to last 72 hours and note the fallback in the briefing.

Apply signal filters per `references/signal-filters.md`.

---

## Step 2 — Filter Rules hard gate (G3)

Before delivering ANY item, check it against `State Dashboard → Filter Rules`. Match by sender, domain, company, or topic → drop silently. No exceptions. Applies to every section: Top 3, Messages, Carryover, Recommended Actions, Risks/Blockers, Watch List Check. A filter rule added after an item was first surfaced still kills it on the next pass.

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

1. **Alerts** — system-level only: backup (Axiom 1), structure drift (Axiom 2), connector failures. NOT personal watch-list items.
   - If backup is flagged: pull free slots from the calendar and propose a specific time. ("You're free tomorrow 2–3 PM. Want me to book a backup setup block then?") If yes, create the calendar event.
   - Q: "Want me to handle any of these?"

2. **Carryover** — unfinished items from the last briefing.
   - Q: "Still relevant, or drop them?"

3. **Schedule** — today's and tomorrow's calendar. Meetings, conflicts, tight windows.
   - Q: "Anything to add or move?"

4. **Top 3** — three most important items for today. Rank: deadline > people waiting > revenue > strategic > other.
   - Q: "Want me to kick any of these off?"

5. **Tasks** — tasks with Status = to-do, in-progress, or blocked. Overdue first, then due-today, then blocked.
   - Q: "Any status changes?"

6. **Messages** — emails, DMs, items needing reply or decision. Sender, subject, one-line summary.
   - Q: "Want me to draft a reply to any of these?"

7. **Missed Message Check** — review `State Dashboard → Missed Messages`. Confirm past corrections still honored. Surface patterns flagged multiple times (candidate for permanent filter rule).
   - Q: "Anything I'm still filtering that you want surfaced?"

8. **Watch List** — items from `State Dashboard → Watch List` needing attention today.
   - Q: "Any updates on these?"

9. **Risks / Blockers** — things that could become problems if ignored (deadlines drifting, budget caps nearing, someone waiting too long).
   - Q: "Anything you want to act on?"

10. **Recommended Actions** — specific next steps. "Reply to [person] about [topic]" not "consider following up."
    - Q: "Want me to queue any of these?"

11. **File Cabinet Check** — soft, optional nudge surfacing records without a full parent chain (tasks without a project, projects without a department, departments without a domain). Render ONLY if there is at least one unnested record. Never frame as a violation. Always offer — don't demand. If user wants to home some, hand off to `flows/file-cabinet.md`.
    - Q: "Want to home any of these now, or leave them for later?"

12. **Rollout Nudge** — one soft sentence per the Pacer pattern (`references/rollout-reminder.md`). Render ONLY when ALL are true: `next_eligible ≤ today`, no Axiom 1 or 2 flag this session, nudge hasn't fired this session. If the render gate fails, skip silently (don't even announce as empty).
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
- **Captain's Log moments** (an interesting idea, a product worth exploring, a lesson learned, a personal win, something worth remembering but not tied to a specific task) → **never auto-write.** Explain why it seems like a Captain's Log entry and let the user decide. ("That five-star review feels like something worth keeping in your Captain's Log as a Win — easy to forget, good to look back on. Want me to log it?")
- **Judgment calls** (not sure where it goes) → offer at the end of the briefing. Keep to 3 items max: "Spotted a few things that might be worth capturing — [short list with where each would go]. Want me to handle any of these?"

For all logging routing details see `flows/documentation-routing.md`.

---

## Step 6 — Housekeeping (after the briefing)

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
