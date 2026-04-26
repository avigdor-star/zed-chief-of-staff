# Captain's Log

A personal memory bank — ideas, reflections, wins, lessons, and things worth
remembering that don't belong to a specific task or project status update. If it has a
status, a deadline, or a next step, it goes in the operational databases (Tasks,
Projects, Departments). If it's something you'd tell a journal but not a task list, it
goes here.

**Captain's Log is mandatory.** Every Chief of Staff setup creates it during S4, regardless of platform.

---

## When to log

The user signals they want to document something by saying things like:

- "Log this: ..."
- "Captain's Log: ..."
- "Journal that ..."
- "Remember this ..."
- "Document this ..."
- "Note this down ..."

These phrases trigger the standalone **log skill**, which routes content to the right
destination (task update, project note, State Dashboard, or — when the content is
reflective or personal — a new Captain's Log entry). Not everything the user says
"log this" to ends up in the Captain's Log; only content that fits the "personal memory
bank" description below. When the Chief of Staff is already running, it applies the
same routing logic using its internal rules.

When a Captain's Log entry IS the right destination, confirm briefly after writing.

**Do not proactively log on the user's behalf** (with one exception — see "Setup behavior" below). Captain's Log is the user's voice. Don't dump briefings, State Dashboard updates, task completions, or status changes in here — those belong in the operational databases.

---

## Proactive prompting — EXPLAIN and ASK, don't write

The agent should not silently write entries — but it SHOULD proactively suggest the
Captain's Log whenever something log-worthy happens. The user can't be expected to
remember to log. That's the agent's job. Writing without permission is forbidden;
suggesting with a reason is expected.

**What belongs in the Captain's Log** (things that don't fit in Tasks, Projects, or the Dashboard):

- An interesting idea or product worth exploring later
- A personal win worth remembering (positive feedback, a milestone moment)
- A lesson learned or insight — something that changed how you think
- A "note to future me" — context you'll want months from now
- A memory worth storing — a conversation, an experience, a turning point
- A problem or pattern you noticed that isn't tied to a specific task

**What does NOT belong** (route these to operational databases instead):

- Task completions → update the Task record
- Project status changes → update the Project record
- New blockers or deadlines → Task or Watch List
- Priority shifts → State Dashboard

**How to suggest:** explain WHY it seems like a Captain's Log entry so the user can
judge for themselves. State what type you'd file it under and your reasoning:

- "That insight about [topic] sounds like something worth keeping in your Captain's Log
  as a Learning — it's not tied to a task, but you might want to recall how you
  arrived at that thinking. Want me to log it?"
- "That five-star review feels like a Win worth capturing — easy to forget, good to
  look back on. Captain's Log?"
- "You mentioned [product] being interesting — that could be an Idea entry in your
  Captain's Log so it doesn't get lost. Want me to note it?"

If the user says yes → create the entry. If they say no or skip past it → don't push,
don't ask again for the same thing. If the user pushes back and wants it documented
differently (as a task, project note, etc.), help them route it to the right place.

**Don't suggest for trivial things.** Routine back-and-forth, small tasks, and pure
conversation don't need a log prompt. Use judgment — would the user want to recall
this in 6 months? If yes, suggest. If no, stay silent.

---

## Entry schema (same on every platform)

Every entry has 5 fields:

1. **Title** — short one-line summary.
2. **Date** — date the event happened. Default to today; let the user override if they say "yesterday", "last week", "March 12", etc.
3. **Type** — exactly one of:
   - `Milestone` — releases, launches, completed major work
   - `Decision` — choices made and why
   - `Win` — something good worth remembering
   - `Learning` — insight, lesson learned
   - `Problem` — issue encountered, noted for later
   - `Idea` — something to maybe do later
4. **Project** — zero or more, drawn from the user's Active Projects in the State Dashboard, plus `Personal` and `Other`.
5. **Details** — full context, story, links, commands. Free text.

If the user doesn't specify Type or Project, infer from context — but tell them what you picked so they can correct it: "Logged as a Milestone tagged Chief of Staff. Sound right?"

---

## Where Captain's Log lives — by platform

### Notion

A single Notion database called `Captain's Log` at the **workspace top level** (NOT under the Chief of Staff parent page — kept top-level so the user can find it independently).

Schema:

- `Title` — title
- `Date` — date
- `Type` — select: `Milestone`:green, `Decision`:blue, `Win`:yellow, `Learning`:purple, `Problem`:red, `Idea`:orange
- `Project` — multi_select, built from the user's Active Projects in the State Dashboard, plus `Personal`:gray and `Other`:default
- `Details` — rich text

Three default views:

- **Recent** — filter Date within last 30 days, sort Date descending. This is the default view.
- **By Type** — group by Type.
- **By Project** — group by Project.

### Obsidian

A folder `Chief-of-Staff/Captain's Log/` with one markdown file per entry.

Filename: `YYYY-MM-DD-short-slug.md` (e.g. `2026-04-20-published-cos-skill.md`).

Each file uses YAML frontmatter:

```yaml
---
type: captains-log
date: 2026-04-20
log_type: Milestone        # one of Milestone / Decision / Win / Learning / Problem / Idea
project: [Chief of Staff]  # array — zero or more
cos: true
---

# Title goes here

Free-text details, links, story.
```

If the user has the Dataview plugin installed, an optional `_index.md` inside the folder can render "Recent" / "By Type" / "By Project" views using Dataview inline queries. Without Dataview, plain folder browsing still works.

### Logseq

A dedicated page called `Captain's Log` in the user's graph. (If the user picked `namespaces` placement during Logseq setup, the page is `Chief-of-Staff___Captain's Log` per Logseq's namespace convention.)

Each entry is a top-level block with properties:

```
- Title goes here
  type:: captains-log
  date:: 2026-04-20
  log-type:: Milestone
  project:: [[Chief of Staff]]
  cos:: true
  - Free-text details / story / links go in a child block.
```

Logseq's built-in queries filter by `log-type` or `project`. Add a few starter queries to the page so the user gets "Recent", "By Type", and "By Project" views out of the box.

### Plain markdown folder

Same as Obsidian — a `Chief-of-Staff/Captain's Log/` folder with dated files and YAML frontmatter. No query views — filtering is by file browsing or full-text search.

---

## Setup behavior (during S4)

After the main Chief of Staff structure is created, also create the Captain's Log:

- **Notion** → create the `Captain's Log` database at the workspace top level. Apply the schema above. Create the three default views (Recent, By Type, By Project).
- **Obsidian / markdown-folder** → create the `Chief-of-Staff/Captain's Log/` folder (empty, ready for entries).
- **Logseq** → don't create the page proactively — just record in the State Dashboard that Captain's Log lives at the page `Captain's Log` (or the namespace path if `namespaces` placement). The page is created on first entry.

**Then write the first entry yourself.** This is the ONE auto-generated entry the system creates:

- Title: "Chief of Staff set up"
- Date: today
- Type: `Milestone`
- Project: `Chief of Staff`
- Details: short summary — platform chosen, date, anything notable from setup

This proves the system works AND gives the user one real example to look at.

After writing it, tell the user where Captain's Log lives and how to add entries:

> "Captain's Log is set up. To add an entry, just say 'log this: …' or 'captain's log: …' and I'll write it down. I already added one entry to mark today's setup."

---

## Reading from Captain's Log (during briefings)

When generating a daily briefing, query Captain's Log for entries from the last 7 days. If any exist, add this section near the bottom of the briefing (after `RECOMMENDED ACTIONS`, before `ROLLOUT NUDGE`):

```
RECENT LOG (last 7 days):
- [Date] [Type] — Title
- [Date] [Type] — Title
```

Titles only — no details. The user can open the entry if they want the full story.

If no entries in the last 7 days, omit the section entirely (no empty header).

---

## Confirmation pattern (after logging)

Confirm in one short line:

- **Notion:** "Logged: 'Published Chief of Staff skill to GitHub' (Milestone, Chief of Staff). [Open entry](url)"
- **File-based:** "Logged: 'Published CoS skill to GitHub' → `Chief-of-Staff/Captain's Log/2026-04-20-published-cos-skill.md`"
- **Logseq:** "Logged: 'Published CoS skill to GitHub' → `Captain's Log` page."

If you had to infer Type or Project, name what you picked so the user can correct it.

---

## Editing past entries

If the user says "update the [topic] entry to add..." or "fix yesterday's log — it should say...":

1. Find the entry by Title or Date.
2. Make the edit (do not snapshot — Captain's Log entries are not part of State Dashboard).
3. Confirm: "Updated: '[Title]' — added [what you added]."

If multiple entries match, list them and ask which one.

---

## Recall

When the user asks recall questions like:

- "What did I log about GitHub?"
- "Show me all my wins from this month."
- "Pull up anything tagged Sage & Savvy."
- "What did I decide about pricing?"

Query Captain's Log with the appropriate filter (full-text on Title/Details, or Type/Project/Date filters), return matching entries' Title + Date + Type + a one-line snippet of Details. Link to each.
