# Flow: Task Management

**What this flow does:** Add tasks, mark done, query tasks, move between projects, query blocked items.

**When to load it:** User says "add a task," "remind me to," "I need to," "mark X as done," "I finished," "what's on my plate," "what tasks," "what's blocked," or "move [task] to [project]."

**Prerequisites:** Bootstrap complete. Universal Gates G1-G8 from SKILL.md apply.

---

## Task creation

When the user says "Add a task: …" / "Remind me to …" / "I need to …":

1. **Infer from context:** project, priority, due date, goal.
2. **Tell the user what you picked** so they can correct it: "Created task: 'Reply to vendor quote' under [Project] → Marketing, priority high, due Friday. Sound right?"
3. **If no matching project exists** → offer to create one (and link to the right department). If the structure is incomplete (no department, no domain), hand off to `flows/file-cabinet.md` to nest it properly.

**Where the task goes:**
- File-based: create one markdown file in `Chief-of-Staff/Tasks/[task-slug].md` with the task frontmatter from `references/file-based-conventions.md`.
- Notion: create one row in the Tasks database with properties from `references/notion-conventions.md`.

---

## Task queries

### "What's on my plate?" / "What tasks do I have?"

Query tasks where Status != `done` AND Status != `cancelled`. Sort by Due Date ascending. Group by Project.

### "What's blocked?"

Query tasks where Status = `blocked`. Show their `Waiting On` values.

### "What did I finish today/this week?"

Query tasks where Status = `done` and the status change was within the date range.

---

## Task status updates

- "Mark [task] as done" / "I finished [task]" → update Status to `done`. Confirm briefly.
- "[Task] is blocked" → update Status to `blocked`, ask what it's waiting on, fill `Waiting On` field.
- "[Task] is in progress" → update Status to `in-progress`.
- "Cancel [task]" / "Drop [task]" → update Status to `cancelled` (don't delete — keeps history).

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

Tasks live in their own folder (file-based) or database (Notion) and are NOT part of the State Dashboard. Snapshot gate G2 does NOT apply to plain task creation/edits. It DOES apply if the user's request also changes State Dashboard content (e.g., "add to watch list while you're at it").
