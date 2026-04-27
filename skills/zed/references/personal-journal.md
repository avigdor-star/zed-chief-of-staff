# Personal Journal

A personal reflection space — how you're feeling, what you're processing, the
shape of your weeks and seasons. Distinct from operational data (Tasks /
Projects / Dashboard) and from working docs (Supporting Documents).

If it has a status, a deadline, or a next step → operational databases.
If it's a working doc tied to a project (brainstorm, prep, plan, persona,
framework) → Supporting Documents.
If it's about how YOU are doing — mood, energy, wins, challenges, gratitude,
free reflection — it goes here.

**Personal Journal is auto-created on first use.** Setup does NOT create it
proactively. The first time the user engages with the end-of-brief Personal
Check-in section, or says a journal phrase, the Chief creates the journal in
the user's chosen platform (after explicit approval per G1) and writes the
first entry.

> **Replaces Captain's Log (v3).** Personal Journal is the v3 successor to
> Captain's Log. Existing Captain's Log entries are preserved by the v2→v3
> migration in `flows/version-migration.md` (move-all archives them, leave-in-place
> keeps them readable, walking goes one-by-one). The Captain's Log structure stays
> referenced in Stable Names so existing user data is never orphaned.

---

## When to journal

The user signals they want to journal by saying things like:

- "Personal check-in"
- "Journal entry for today"
- "I want to journal about ..."
- "How I'm feeling today"
- "Personal journal: ..."
- "My mood today"

These phrases route to `flows/journal.md`. The end-of-brief Personal
Check-in section ALSO routes there if the user engages.

The general "log this / remember this / document this" phrases stay with
`flows/documentation-routing.md` and ALWAYS ask the user where the content
goes (Task / Project / Dashboard / Personal Journal / Supporting Document).
No silent default.

---

## Proactive prompting — only the end-of-brief check-in

Unlike Captain's Log (which had in-brief proactive prompts during Step 5
"Proactive documentation"), the Personal Journal is offered ONCE per brief —
in the dedicated **Personal Check-in** section at the end of every briefing.

The Chief does NOT scatter "want to journal that?" prompts throughout the
brief. The check-in section is the single, predictable touchpoint.

**Why:** Journal fatigue is a real thing. One predictable prompt per brief is
sustainable; multiple in-brief nudges feel naggy.

**Soft mute.** If the user skips the check-in section 5 briefings in a row,
the Chief asks once: "Want me to dial this back to once a week, or pause it
entirely?" Tracked in session memory across the most recent briefings (use
the State Dashboard `journal_check_in_skips` counter — see Stable Names).

---

## Entry schema (same on every platform)

Every entry has 7 fields:

1. **Date** — date the entry is for. Default to today.
2. **Mood** — one short word or short phrase. Examples: `calm`, `anxious`,
   `excited`, `overwhelmed`, `content`. Free text — no fixed list.
3. **Energy** — one of: `low`, `medium`, `high`. Optional. (Adjective scale,
   not a number, to keep entries human.)
4. **Wins** — what went well. Free text. Optional.
5. **Challenges** — what was hard. Free text. Optional.
6. **Gratitude** — what you're grateful for. Free text. Optional.
7. **Free text** — anything else. The "open page" of the entry. Optional.

All fields except Date are optional. A valid entry can be just Date + a few
words in any one field. The point is sustainability, not completeness.

---

## Where Personal Journal lives — by platform

### Notion

A single Notion database called `Personal Journal` at the **workspace top
level** (NOT under the Chief of Staff parent page — kept top-level so the
user can find it independently, same pattern Captain's Log used).

Schema:

- `Title` — title (auto-set to `YYYY-MM-DD` if user doesn't supply one)
- `Date` — date
- `Mood` — text
- `Energy` — select: `low`:gray, `medium`:yellow, `high`:green
- `Wins` — rich text
- `Challenges` — rich text
- `Gratitude` — rich text
- `Notes` — rich text (the "free text" field; named `Notes` in Notion to fit
  Notion's conventions)
- `cos_id` — text (UUID, written on first read per the v2 cos_id pattern)

Three default views:

- **Recent** — filter Date within last 30 days, sort Date descending. Default view.
- **By Mood** — group by Mood (text-grouped, sorted alphabetically).
- **By Month** — group by Date by month (Notion native grouping).

### Obsidian

A folder `Chief-of-Staff/Personal Journal/` with one markdown file per entry.

Filename: `YYYY-MM-DD.md` (e.g. `2026-04-26.md`). If multiple entries on the
same day are needed (rare), append a short slug: `2026-04-26-evening.md`.

Each file uses YAML frontmatter:

```yaml
---
type: personal-journal
date: 2026-04-26
mood: calm
energy: medium
cos_id: <uuid>
cos: true
---

## Wins
What went well today.

## Challenges
What was hard.

## Gratitude
What I'm grateful for.

## Notes
Free reflection, anything else.
```

Sections are markdown headers, not frontmatter — so they're easy to read in
plain Obsidian without any plugins. Empty sections can be deleted from the
file (no requirement to fill all of them).

If the user has the Dataview plugin installed, an optional `_index.md` inside
the folder can render "Recent" / "By Mood" / "By Month" views using Dataview
inline queries. Without Dataview, plain folder browsing still works.

### Logseq

A dedicated page called `Personal Journal` in the user's graph. (If the user
picked `namespaces` placement during Logseq setup, the page is
`Chief-of-Staff___Personal Journal` per Logseq's namespace convention.)

Each entry is a top-level block with properties:

```
- 2026-04-26
  type:: personal-journal
  date:: 2026-04-26
  mood:: calm
  energy:: medium
  cos-id:: <uuid>
  cos:: true
  - **Wins**
    - What went well today.
  - **Challenges**
    - What was hard.
  - **Gratitude**
    - What I'm grateful for.
  - **Notes**
    - Free reflection.
```

Logseq's built-in queries filter by `mood` or by date range. Add a few
starter queries to the page so the user gets "Recent" and "By Mood" views
out of the box.

### Plain markdown folder

Same as Obsidian — a `Chief-of-Staff/Personal Journal/` folder with dated
files and YAML frontmatter. No query views — filtering is by file browsing
or full-text search.

---

## First-use creation behavior (auto-create on first engagement)

When the user first engages with journaling (either by saying a journal
phrase, or by responding to the end-of-brief Personal Check-in), the Chief
does this once per platform:

1. Check if Personal Journal already exists at the expected location.
2. If yes → skip creation, proceed to entry.
3. If no → tell the user once: "I'll set up your Personal Journal in
   [platform] now — takes a second." Snapshot State Dashboard first per G2
   (because we'll record the journal location in Stable Names afterward).
4. Create the structure per the platform-specific layout above.
5. Write the user's first entry per their engagement (not an auto-generated
   placeholder — Personal Journal entries are the user's voice).
6. Confirm: "Your Personal Journal lives at [location]. From now on, just
   say 'journal entry' or engage the Personal Check-in at the end of any
   brief, and I'll write what you say."

**No auto-generated first entry.** Unlike Captain's Log (which had a
"Chief of Staff set up" auto-entry to prove the system works), Personal
Journal entries are reflective and personal — they should be the user's
words, not the Chief's. The first real entry from the user is enough proof.

---

## Reading from Personal Journal (during briefings)

**Removed from the brief in v3.** The "Recent Log (last 7 days)" section
that Captain's Log used to render is gone. Journal entries are personal —
they don't need to surface in operational briefings.

If the user wants to recall journal entries, they ask via the cross-system
recall flow (`flows/recall.md`), which queries Personal Journal alongside
Supporting Documents and Briefings.

---

## Confirmation pattern (after writing an entry)

Confirm in one short line:

- **Notion:** "Journaled for today (mood: calm, medium energy). [Open entry](url)"
- **File-based:** "Journaled for today → `Chief-of-Staff/Personal Journal/2026-04-26.md`"
- **Logseq:** "Journaled for today → `Personal Journal` page."

If the entry was a quick free-text-only entry, say so: "Quick note logged for
today. Nothing else needed unless you want to add more."

---

## Editing past entries

If the user says "update yesterday's entry to add..." or "fix Monday's mood
— it should say 'tired' not 'calm'":

1. Find the entry by Date.
2. Make the edit (do not snapshot — Personal Journal entries are NOT part of
   State Dashboard, so G2 doesn't apply).
3. Confirm: "Updated yesterday's entry — added [what you added]."

If multiple entries match the date (rare), list them and ask which one.

---

## Recall

When the user asks recall questions like:

- "What did I journal about last week?"
- "Show me when I felt overwhelmed."
- "What was on my mind in March?"
- "Pull up my journal entries from this month."

This is handled by the cross-system recall flow (`flows/recall.md`), which
searches Personal Journal alongside Supporting Documents and Briefings.

---

## Relationship to Supporting Documents (graduation rule)

If a Personal Journal entry's Free Text section grows multi-paragraph AND
ties to an active Project, the Chief can offer to graduate it to a
Supporting Document brainstorm — same pattern that existed for Captain's
Log Idea entries. See `references/documentation-routing.md` § graduation
rule for the mechanics.

The original Personal Journal entry is preserved (with a one-line pointer
to the new Supporting Document). Reflection stays in the journal; project
work moves to the working doc.

---

## G9 PRC summary for Personal Journal

- **Provenance.** Every entry records `Date` (creation), `cos_id` (UUID
  identifier), `mood` if supplied, `energy` if supplied. Created-by is
  always the user (the Chief never auto-writes journal entries).
- **Reversal.** Every entry can be edited or deleted by the user via the
  edit flow above. Graduation to Supporting Document is reversible (delete
  the SD, the original PJ entry remains intact).
- **Conflict-Detection.** The graduation offer is the single conflict point
  — the Chief asks before promoting Idea-flavored journal content to a
  Supporting Document. Otherwise, the journal is the user's voice and the
  Chief never silently moves content out of it.
