# Flow: Task Management

**What this flow does:** Add tasks, mark done, query tasks, move between projects, query blocked items, query awaiting-deploy items.

**When to load it:** User says "add a task," "I need to," "mark X as done," "I finished," "what's on my plate," "what tasks," "what's blocked," "what's awaiting deploy," "shipped X," or "move [task] to [project]." (NOTE: "remind me to / about" routes to `flows/reminders.md` — Reminders are distinct from Tasks per v2.)

**Prerequisites:** Bootstrap complete. Universal Gates G1-G9 from SKILL.md apply.

---

## Task creation

When the user says "Add a task: …" / "I need to …":

1. **Infer from context:** project, priority, due date, **Time Sensitivity** (added v2 — see anchors below), goal.
2. **Time Sensitivity prompt** (added v2): if Time Sensitivity isn't obvious from context, ask using the anchor table:
   > "How time-sensitive is this on a 0–5 scale?
   > 0 = no rush (e.g., 'explore competitor landscape')
   > 1 = soft this-month
   > 2 = soft this-week
   > 3 = soft this-day
   > 4 = hard same-day (e.g., '5pm pickup')
   > 5 = drop-everything-now (hard override)"
   Default to leaving it null/0 if the user doesn't answer — composite ranking treats null as 0.
3. **Fuzzy duplicate check against active Reminders (added v2 — G9 conflict-detection).** Run the same fuzzy match used in `flows/reminders.md` § Fuzzy duplicate check (2+ shared non-stop-word tokens OR Levenshtein ≤ 3, same Project). If a match is found, surface three options: replace / link / separate. "Link" sets the Reminder's `Linked Task` to the new Task's `cos_id`.
4. **Tell the user what you picked** so they can correct it: "Created task: '[name]' under [Project] → [Department], priority high, Time Sensitivity 3, due Friday. Sound right?"
5. **If no matching project exists** → offer to create one (and link to the right department). If the structure is incomplete (no department, no domain), hand off to `flows/file-cabinet.md` to nest it properly.

**Where the task goes:**
- File-based: create one markdown file in `Chief-of-Staff/Tasks/[task-slug].md` with the task frontmatter from `references/file-based-conventions.md`. Generate a UUID for `cos_id`.
- Notion: create one row in the Tasks database with properties from `references/notion-conventions.md`.

---

## Task queries

### "What's on my plate?" / "What tasks do I have?"

Query tasks where Status NOT IN (`done`, `cancelled`, `not-deployed`). Sort by composite rank per `references/signal-filters.md` § Within-category ranking, then by Due Date ascending. Group by Project.

### "What's blocked?"

Query tasks where Status = `blocked`. Show their `Waiting On` values.

### "What did I finish today/this week?"

Query tasks where Status = `done` and the status change was within the date range.

### "What's awaiting deploy?" (added v2)

Query tasks where Status = `not-deployed`, grouped by Project, sorted by last-edited descending. If 5+ items, render as count + collapsed list (matches the briefing's Awaiting Deploy subsection).

---

## Task status updates

- "Mark [task] as done" / "I finished [task]" → update Status to `done`. Confirm briefly.
- "[Task] is blocked" → update Status to `blocked`, ask what it's waiting on, fill `Waiting On` field.
- "[Task] is in progress" → update Status to `in-progress`.
- **"[Task] is built but not yet shipped"** / "ready to deploy [task]" (added v2) → update Status to `not-deployed`. Confirm: "Marked as awaiting deploy. I'll surface it in the Awaiting Deploy subsection until you say it's shipped."
- **"Shipped [task]"** / "deployed [task]" / "[task] is live" (added v2) → if Task is currently `not-deployed`, flip Status to `done`. Confirm. (The Chief never auto-flips — always requires explicit user statement.)
- "Cancel [task]" / "Drop [task]" → update Status to `cancelled` (don't delete — keeps history). **Dangling-link cleanup (added v2):** scan Reminders for `Linked Task = this task's cos_id`. For each match, surface to the user: "Reminder '[reminder name]' was linked to this task. Clear the link, or dismiss the reminder?" See `flows/reminders.md` § Dangling-link cleanup.

---

## Project assignment changes

"Move [task] to [project]" → update the task's Project relation/frontmatter to the target project.

If the target project doesn't exist:
- Offer to create it: "There's no [project name] yet. Want me to create it under [department]?"
- If the user says yes and there's no department either, hand off to `flows/file-cabinet.md`.

---

## Hand-off to file-cabinet

If a task references a non-existent project, department, or domain — or the user mentions a new bucket while creating a task — hand off to `flows/file-cabinet.md` for the nesting offer. Don't force nesting; an unparented task still works and surfaces in briefings (it shows up as a File Cabinet Check nudge later).

---

## Snapshot rule

Tasks live in their own folder (file-based) or database (Notion) and are NOT part of the State Dashboard. Snapshot gate G2 does NOT apply to plain task creation/edits. It DOES apply if the user's request also changes State Dashboard content (e.g., "add to watch list while you're at it"). Reminder dangling-link cleanup writes to Reminders (not State Dashboard), so no G2 there either.
