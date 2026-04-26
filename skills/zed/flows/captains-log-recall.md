# Flow: Captain's Log Recall

**What this flow does:** Queries the Captain's Log for entries matching a user request and returns the matches.

**When to load it:** User asks recall questions like "what did I log about X," "show me my wins this month," "what did I decide about Y," "pull up anything tagged [project]."

**Prerequisites:** Bootstrap complete. Captain's Log exists (created during setup S4.5). Read `references/captains-log.md` for the schema and platform-specific storage details.

---

## Query patterns

Translate the user's question into a query against the Captain's Log:

- **By Type:** "show me my wins" → filter `Type = Win`. "what did I decide about pricing" → filter `Type = Decision` + full-text on Title/Details for "pricing."
- **By Project:** "anything tagged Sage & Savvy" → filter `Project includes Sage & Savvy`.
- **By date range:** "wins this month" → filter `Type = Win` AND `Date within current month`. "what did I log last week" → filter `Date within last 7 days`.
- **By keyword:** "what did I log about GitHub" → full-text search on Title and Details for "GitHub."

If the question combines criteria (e.g., "decisions about pricing this quarter"), AND them together.

---

## Where to query (by platform)

- **Notion:** query the top-level `Captain's Log` database. Use property filters for Type / Project / Date and full-text search on Title / Details for keywords.
- **Obsidian / markdown-folder:** scan files in `Chief-of-Staff/Captain's Log/` — read frontmatter for Type / Project / Date filtering, full-text search on file body for keywords.
- **Logseq:** query the `Captain's Log` page (or namespace path if `namespaces` placement) — filter blocks by `log-type::` / `project::` / `date::` properties, full-text search for keywords.

---

## Result formatting

For each matching entry, return:
- **Title**
- **Date**
- **Type**
- A one-line snippet from Details (first sentence or so — enough to remind the user)

Format as a compact list. Link to the full entry per the platform:
- Notion: `[Open entry](url)` for each row
- File-based: `Chief-of-Staff/Captain's Log/[filename].md`
- Logseq: `Captain's Log` page (or namespace path)

If there are many matches, show the 10 most recent and tell the user "there are [N] total — say 'show all' if you want the rest."

If nothing matches, say so plainly: "Nothing in your Captain's Log matches that. Want me to broaden the search?"

---

## What this flow does NOT do

- Doesn't write to the Captain's Log. Writes are handled by `flows/documentation-routing.md` (when content is routed there) or directly when the user explicitly says "captain's log: …".
- Doesn't snapshot the State Dashboard. Captain's Log is a separate database/folder; G2 doesn't apply to reads.
