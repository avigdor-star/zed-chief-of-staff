# Notion Conventions for Chief of Staff Documents

> **Applies to:** users who picked `notion` during the platform-choice flow (Step S1.5).
>
> **Using Obsidian, Logseq, or plain markdown?** Stop here and load `references/file-based-conventions.md` — the file-based schema is different (folders and frontmatter instead of databases and properties).

This doc is the Notion equivalent of `file-based-conventions.md` — same concepts in Notion's mechanics: databases instead of folders, properties instead of frontmatter, relations instead of `[[wikilinks]]`.

---

## Top-level structure

All Chief of Staff content lives under a single parent page in the user's Notion workspace:

```
Chief of Staff                       ← parent page (top-level)
├── State Dashboard                  ← page (page-level properties: Last Session At, setup_status, Last Seen Version, Migration v2 Choice, Migration v3 Choice, Legacy Plugin Choice, Personal Journal Location, Journal Check-in Skips, Journal Check-in Frequency)
├── Live Feed                        ← page
├── Snapshots                        ← page (holds state-snapshot-* page duplicates)
├── 🌐 Domains                       ← database (top of hierarchy: life entities — e.g., Business A, Personal)
├── 🏢 Departments                   ← database (functional areas within a domain — e.g., Marketing, Health)
├── 🚀 Projects                      ← database (specific initiatives within a department)
├── ✅ Tasks                          ← database (gains Time Sensitivity property + not-deployed Status option in v2)
├── 📚 Briefings                     ← database
├── 📂 Supporting Documents          ← database (added v2 — absorbs Research / Drafts / Action Plans / Reference via Type field)
├── ⏰ Reminders                     ← database (added v2 — date-scoped surfacing nudges, parallel to Tasks)
├── 👤 People                        ← database
├── 🔍 Research                      ← database (LEGACY in v2 — read-only; new entries go to Supporting Documents)
├── ✉️ Drafts                         ← database (LEGACY in v2 — read-only; new entries go to Supporting Documents)
├── 📋 Action Plans                  ← database (LEGACY in v2 — read-only; new entries go to Supporting Documents)
├── 📎 Reference                     ← database (LEGACY in v2 — read-only; new entries go to Supporting Documents)
└── 📁 Archive Views (optional)      ← page of linked views showing `Status = archived` across databases

Workspace top level (NOT under Chief of Staff parent — kept independent):
├── 🪞 Personal Journal              ← database (added v3 — auto-created on first journal use; mood / energy / wins / challenges / gratitude / notes)
└── 📓 Captain's Log                 ← database (LEGACY in v3 — read-only; preserved for users with v1/v2 entries; new reflective content goes to Personal Journal)
```

**Legacy DB read rule.** The four legacy databases (Research / Drafts / Action Plans / Reference) are still read by briefing source-scan and recall queries — entries are tagged `[legacy]` in surfaced output. New entries always go to Supporting Documents. See `references/documentation-routing.md` § Legacy DB read rule.

**Three things called "archive" / "snapshot" — keep straight:**
- `Status = archived` on a database record → cleanup/lifecycle state (see "Archive strategy").
- `📁 Archive Views` subpage → a view, not a container (records don't move there).
- `Snapshots` subpage → where page-level duplicates of State Dashboard live for rollback.

The Chief of Staff creates this tree during Step S4 of SKILL.md. The Notion connector lets it create pages and databases programmatically.

**Emoji prefixes** on database names are optional — they help scannability in Notion's sidebar. Drop them if the user prefers plain names.

---

## State Dashboard (page, not a database)

The State Dashboard is a single Notion page — editable, section-based, mirroring the markdown version.

### Structure

Use Notion headings + callouts + embedded database views. The page is laid out as:

- **# Chief of Staff — State Dashboard** (H1)
- **Page-level properties** (set on the page itself, not in a sub-table — these are accessed as Notion page properties so the Chief and watchers can read them via the API without parsing the page body):
  - `Last Session At` (Date) — set by the Chief at the end of every session. The "since last session" anchor for the Monday rule. Do NOT rely on Notion's built-in `Last edited` field — it bumps every time anything writes to the page (including watchers, the Health Check, the Weekly Review, manual edits) and would give a wildly wrong window.
  - `setup_status` (Select: `in_progress` / `complete` / `complete-with-warning`) — set during setup. `complete-with-warning` means S6 backup was deferred and Axiom 1 should keep nudging.
  - `Last Seen Version` (Text, added v2) — the canonical version the user last completed migration through (e.g., `v1`, `v2`). Bootstrap step 3 compares this to SKILL.md's `Current:` value.
  - `Migration v2 Choice` (Select, added v2): `moved` / `leave-in-place` / `leave-permanent` / `walking` / `complete`. Tracks what the user picked when prompted by `flows/version-migration.md`.
  - `Legacy Plugin Choice` (Select, added v2): `uninstalled` / `dismissed` / `permanent`. Tracks what the user picked for the cross-plugin detection prompt.
- **## My Setup** → a simple two-column table (or a set of inline mentions). Fields: Name, Email, Role, Company/Team, Chief Name, **Personality** (one of: `Professional`, `Playful & lighthearted`, `Dry wit`, `Warm & encouraging`, or a custom description; defaults to `Professional` if blank), Alert Threshold, **Platform** (always `notion` for this schema). Mirror copies of the new v2 fields above can also live here for human visibility, but the page-level properties are the authoritative source the Chief reads.
- **## Active Projects** → an embedded linked view of the Projects database, filtered to `Status = active`, grouped by Department.
- **## Tasks Due** → an embedded linked view of the Tasks database, filtered to `Status != done AND Status != cancelled`, sorted by Due Date ascending. Shows what's on the plate right now.
- **## High-Priority People** → an embedded linked view of the People database, filtered to `Priority = high`.
- **## Watch List** → a bulleted list inside a toggle block. Each bullet is a short item. (A database is overkill here.)
- **## Missed Messages** → bulleted list inside a toggle.
- **## Open Decisions** → bulleted list inside a toggle.
- **## Surfaced Items** (added v2) → an inline table tracking briefing items the user has marked. Columns: `Identifier` (canonical ID per `references/signal-filters.md`), `Surfaced On` (date), `State` (select: `active` / `suppress-forever` / `suppress-this-thread` / `handled`), `Source` (which brief section it came from), `Notes`. Default state for new entries is `active`. Items in any non-active state are dropped from briefings (G3 + Surfaced Items state check). `handled` items archive after 30 days.
- **## Filter Rules** (existing — gains v2 columns) → table of patterns to silently drop. Columns: `Pattern`, `Match Type` (sender / domain / company / topic), `Source` (added v2 — Select: `manual` / `suppress-forever` / `setup` / `migration`), `Created At` (added v2 — Date). The `Source` enum lets the user audit and reverse Filter Rules created via "suppress forever" using the un-suppress verb (see `flows/interactive.md`).
- **## This Week's Focus** → free text, updated weekly.
- **## Weekly Trends** → free text, auto-updated by the Weekly Review routine.
- **## Automation Status** → a simple inline table with Phase / Week / Component / Status columns. (Not a database — this never grows.)
- **## Check-in State** → a simple inline two-column table (Field / Value) holding the Pacer state. Not a database — small, fixed schema. Fields: Last Mentioned, Last Contextual Mention, Next Eligible, Last Check-in Response, Last Action, Express Partial, Express Installed. See `references/rollout-reminder.md` for field semantics.
- **## System Health** → callouts with Desktop Tasks, Cloud Routines, and Connectors status. Same tables as in the markdown version.

### Check-in State table (literal layout)

Create this table under the `## Check-in State` heading during setup. Leave values blank except `Next Eligible`, which is set to today + 7 days at the end of S4.

| Check-in State Field | Value |
|------------------------|-------|
| Last Mentioned | *(blank until first nudge fires)* |
| Last Contextual Mention | *(blank until first contextual opening fires)* |
| Next Eligible | YYYY-MM-DD *(today + 7 days at setup)* |
| Last Check-in Response | *(blank — one of: overwhelming / just_right / ready_for_more)* |
| Last Action | *(blank — one of: snooze / express)* |
| Express Partial | false |
| Express Installed | *(blank — comma-separated list once populated)* |

Updates to this table are made in place by the Chief of Staff during sessions, following the write rules in `references/rollout-reminder.md`.

### Snapshot rules and restoration

Snapshot mechanics, the snapshot-first hard gate, and restoration procedures all live in `references/vault-safety.md`. One Notion-only specific that doesn't fit there: during S4, create a `Snapshots` subpage directly under the Chief of Staff parent page (sibling to State Dashboard, Live Feed, and the seven databases) — that's where page-snapshots are stored, kept separate from Archive (a `Status = archived` filter on database records — see "Archive strategy" below).

---

## Live Feed (page, not a database)

The Live Feed is also a single page, structured as an appending log. Watchers add entries to the bottom. The Chief of Staff clears handled entries.

### Structure

- **# Chief of Staff — Live Feed** (H1)
- **## Urgent Alerts** — callout block at the top. Empty by default.
- **## Feed** — the running log. Watchers append under here.
- Each entry is a toggle heading with the format: `### [TIMESTAMP] — [Type] — [Short summary]`. Inside the toggle: sender, subject, summary, urgency, status.
- Handled entries get a `✅` prefix on their heading. Carried-over entries get a `↪️` prefix.

### Why a page and not a database

A Notion database for feed entries would be cleaner in many ways (filterable by status, sortable by timestamp). The downside: watchers would have to create a new database row for every append, which is heavier and slower via the API. A page with a running log matches the markdown Live Feed closely and is simpler for automated watchers.

**If the user prefers a Live Feed database** — that's also fine. Create it with properties: Timestamp (date), Type (select), Sender (text), Subject (text), Summary (text), Urgency (select), Status (select: new / handled / carried-over). Surface this option during setup if the user asks about it.

---

## Databases

Each database below mirrors a file-based folder. Properties replace frontmatter. Relations replace `[[wikilinks]]`.

### Briefings database

Mirrors `Briefings/` folder.

| Property | Type | Purpose |
|----------|------|---------|
| Name | Title | Briefing title (e.g., "Briefing — YYYY-MM-DD", "Weekly Review — YYYY-MM-DD") |
| Date | Date | The date of the briefing |
| Type | Select | `briefing` / `weekly-review` / `monthly-cleanup` |
| Sources | Multi-select | `email`, `calendar`, `tasks`, `messaging` |
| Related Projects | Relation → Projects | Projects this briefing touched |
| Related People | Relation → People | People referenced |
| Previous Briefing | Relation → Briefings | Self-relation to yesterday's briefing |
| Status | Select | `active` / `archived` |

**Body of each record:** the full briefing content — Yesterday's Carryover, Today's Top 3, Schedule, Messages, Watch List, Risks, Recommended Actions.

### Research database (LEGACY in v2 — read-only)

> **Status:** Legacy. v2 absorbed Research into Supporting Documents (Type = `analysis`, Tag = `research`). This database is read by briefing source-scan and recall — entries are tagged `[legacy]` in surfaced output. Do NOT create new entries here in v2 — route to Supporting Documents instead. See `references/documentation-routing.md` § Legacy DB read rule.

Mirrors `Research/` folder.

| Property | Type | Purpose |
|----------|------|---------|
| Name | Title | Topic (e.g., "Competitor Pricing") |
| Date | Date | When researched |
| Requested By | Relation → Briefings | The briefing that kicked off this research |
| Related Projects | Relation → Projects | |
| Status | Select | `active` / `archived` |

**Body:** Question, Findings, Recommendation.

### Drafts database (LEGACY in v2 — read-only)

> **Status:** Legacy. v2 absorbed Drafts into Supporting Documents (Type = `artifact`, Tag = `draft`). Read-only as above.

Mirrors `Drafts/` folder.

| Property | Type | Purpose |
|----------|------|---------|
| Name | Title | Recipient or topic |
| Date | Date | When drafted |
| For | Relation → People (or text if unknown) | Who it's going to |
| Regarding | Text | Subject / topic |
| Related Briefing | Relation → Briefings | The briefing that requested the draft |
| Status | Select | `draft` / `sent` / `discarded` |

**Body:** the actual draft text.

### Action Plans database (LEGACY in v2 — read-only)

> **Status:** Legacy. v2 absorbed Action Plans into Supporting Documents (Type = `plan`, Tag = `action-plan`). Read-only as above.

Mirrors `Action-Plans/` folder.

| Property | Type | Purpose |
|----------|------|---------|
| Name | Title | Plan title |
| Date | Date | When created |
| Project | Relation → Projects | Which project it belongs to |
| Related Briefing | Relation → Briefings | |
| Status | Select | `active` / `completed` / `archived` |

**Body:** Goal, Steps, Timeline, Dependencies / Risks.

### Domains database

Top of the file cabinet (four-level hierarchy). A domain is a distinct life entity — a business, "Personal," or a shared/admin bucket. Most users have 3–5 domains.

| Property | Type | Purpose |
|----------|------|---------|
| Name | Title | Domain name (e.g., "My Agency", "Personal", "Shared") |
| Description | Text | One-line summary of what this domain covers |
| Status | Select | `active` / `archived` |

**Body:** Optional — notes, mission statement, or links to external resources for this domain.

### Departments database

Second level. A department is a functional area within a domain — the kind of work, not a specific initiative. Examples: Marketing, Product, Finance, Health, Family, Home Maintenance.

| Property | Type | Purpose |
|----------|------|---------|
| Name | Title | Department name (e.g., "Marketing", "Health") |
| Domain | Relation → Domains | Which domain this belongs to |
| Description | Text | What this department covers |
| Status | Select | `active` / `archived` |

**Body:** Optional — ongoing notes, KPIs, or links relevant to this area.

### Projects database

Mirrors `Projects/` folder. Third level — a specific initiative within a department.

| Property | Type | Purpose |
|----------|------|---------|
| Name | Title | Project name |
| Department | Relation → Departments | Which department this belongs to |
| Status | Select | `active` / `paused` / `completed` |
| Started | Date | Kickoff date |
| Current Phase | Text | Short status |
| Next Milestone | Text | Short text |
| Blockers | Text | What's holding progress |
| Notes | Text | Ongoing notes |
| Key People | Relation → People | |
| Related Briefings | Relation → Briefings (auto back-relation) | |

**Body:** long-form notes, decisions, links to external files.

### Tasks database

Fourth level — the actual to-dos. Every task belongs to a project. Through that project, it inherits a department and domain.

| Property | Type | Purpose |
|----------|------|---------|
| Name | Title | Task name — a clear action (e.g., "Reply to vendor quote", "Schedule annual physical") |
| Status | Select | `to-do` / `in-progress` / `blocked` / `done` / `cancelled` / **`not-deployed`** (added v2) |
| Priority | Select | `urgent` / `high` / `medium` / `low` |
| Due Date | Date | When it's due (optional — not every task has a deadline) |
| **Time Sensitivity** (added v2) | Number | 0–5 scale for intuitive urgency. See `references/signal-filters.md` for anchors and the composite ranking formula. |
| Project | Relation → Projects | Which project this belongs to |
| Goal | Text | Optional — what outcome this task serves (for alignment without adding a 5th hierarchy level) |
| Notes | Text | Context, details, links |
| Waiting On | Text | Who or what is blocking this (only relevant when Status = blocked) |
| Related Briefing | Relation → Briefings | The briefing that spawned this task (if applicable) |

**Body:** Optional — checklists, sub-steps, or detailed notes. Use Notion's built-in checklist blocks for sub-dividing complex tasks rather than creating a 5th hierarchy level.

**Saved view audit (v2 — required by G7' for the new Status option `not-deployed`):**
- Default "Active Tasks" view filter: `Status != done AND != cancelled` → **update to also exclude `not-deployed`**.
- "Tasks Due" linked view on State Dashboard: same filter → **same update**.
- New default view "Awaiting Deploy": filter `Status = not-deployed`, sort by `Last Edited` descending. Used by briefing's Awaiting Deploy subsection.
- Any user-created views filtering on Status: surface during migration ("I see N saved views filter on Task Status — do they need updating? [list]"). Document touched views in CHANGELOG.

### Supporting Documents database (added v2)

Absorbs the four legacy databases (Research / Drafts / Action Plans / Reference) into one. Distinguished by Type. Sits sideways from the four-level hierarchy — relates to Projects / Tasks / Departments via relations, not nested under any single one.

| Property | Type | Purpose |
|----------|------|---------|
| Name | Title | Document name |
| Type | Select | `brainstorm` / `prep` / `analysis` / `plan` / `artifact`. See `references/documentation-routing.md` for the decision tree. |
| Date | Date | When the doc was created or last meaningfully edited |
| Tags | Multi-select | Free-form, normalized to canonical list per `references/documentation-routing.md`. Suggested seeds: `persona`, `framework`, `worksheet`, `draft`, `research`, `reference`, `action-plan`, `meeting-prep`, `competitor`, `archived`, `legacy-uncategorized` |
| Related Projects | Relation → Projects | |
| Related Tasks | Relation → Tasks | |
| Related Departments | Relation → Departments | |

**No Status property.** Use `Tag = archived` to retire a doc (briefing source-scan and recall queries exclude `Tag = archived`).

**Body:** the actual document content — long-form notes, frameworks, prep notes, drafts, etc.

**Saved view recommendations (default views to create at S4):**
- "All Active" — filter `Tag != archived`, default sort by Date descending.
- "By Type" — group by Type.
- "By Project" — group by Related Projects.

### Reminders database (added v2)

Date-scoped surfacing nudges. Distinct from Tasks (work to do). Sits sideways — can attach to Project / Department / Domain, or be orphaned. Surfaces in the briefing's "Reminders Today" section.

| Property | Type | Purpose |
|----------|------|---------|
| Name | Title | Reminder name |
| surface_on | Date | The day the brief should show this. Recurring reminders advance per terminal-action rule (see `flows/reminders.md`). |
| Recurrence | Select | `none` / `daily` / `weekly` / `monthly` / `quarterly` / `yearly`. (Custom recurrence deferred to v2.1.) |
| Project | Relation → Projects | Optional |
| Department | Relation → Departments | Optional |
| Domain | Relation → Domains | Optional |
| Notes | Text | |
| Status | Select | `active` / `snoozed` / `dismissed` |
| snoozed_until | Date | Null when not snoozed |
| Source Item | Text | Canonical identifier (per `references/signal-filters.md`) when the reminder was created from a Surfaced Items "remind me later" — links the reminder back to the original email/message/task. Null otherwise. |
| Linked Task | Relation → Tasks | Optional. Set by the fuzzy-match dedup at create time. Notion's auto back-relation provides reverse-lookup. |
| Priority | Select | `high` / `medium` / `low`. Defaults `medium`. Used for sort within "Reminders Today." |

**Saved view recommendations (default views at S4):**
- "Reminders Today" — filter `Status = active AND surface_on <= today AND (snoozed_until is empty OR snoozed_until <= today)`, sort overdue first then by Priority.
- "Snoozed" — filter `Status = snoozed`.
- "Dismissed" — filter `Status = dismissed` (history view).
- "Upcoming (7 days)" — filter `Status = active AND surface_on between today and today+7d` (used by lookahead).

### Personal Journal database (added v3 — workspace top level, NOT under CoS parent)

Personal reflection space. Auto-created on first use (NOT during setup S4). Located at workspace top level so the user can find it independently — same pattern Captain's Log used in v1/v2. Schema and full storage details in `references/personal-journal.md`.

| Property | Type | Purpose |
|----------|------|---------|
| Title | Title | Auto-set to date string if blank (e.g., `2026-04-27`) |
| Date | Date | Date the entry is for (default today) |
| Mood | Text | Free text — one word or short phrase. No fixed list. |
| Energy | Select | `low`:gray / `medium`:yellow / `high`:green. Optional. |
| Wins | Rich Text | What went well. Optional. |
| Challenges | Rich Text | What was hard. Optional. |
| Gratitude | Rich Text | What user is grateful for. Optional. |
| Notes | Rich Text | Free-text "open page." Optional. |
| cos_id | Text | UUID for cross-system identity (per v2 cos_id pattern). |

**Saved view recommendations (default views, created on auto-create at first use):**
- "Recent" — filter Date within last 30 days, sort Date descending. Default view.
- "By Mood" — group by Mood (text-grouped, sorted alphabetically).
- "By Month" — group by Date by month (Notion native grouping).

**Briefing source-scan rule:** Personal Journal is NOT scanned during briefings. Recall via `flows/recall.md` if needed.

### Captain's Log database (LEGACY in v3 — workspace top level, read-only)

Preserved for users with v1/v2 entries. New reflective content goes to Personal Journal (auto-created on first use). Schema unchanged from v2:

| Property | Type | Purpose |
|----------|------|---------|
| Title | Title | |
| Date | Date | |
| Type | Select | `Milestone` / `Decision` / `Win` / `Learning` / `Problem` / `Idea` |
| Project | Multi-select | Active Projects from Dashboard, plus `Personal` and `Other` |
| Details | Rich Text | |

**Read-only in v3.** Recall queries (`flows/recall.md`) still surface entries here, tagged `[legacy CL]`. The v2→v3 migration in `flows/version-migration.md` offers move-all (archive) / leave-in-place / walking.

### People database

Mirrors `People/` folder.

| Property | Type | Purpose |
|----------|------|---------|
| Name | Title | First Last |
| Relationship | Select | `client` / `collaborator` / `vendor` / `family` / `other` |
| Priority | Select | `high` / `medium` / `low` |
| Email | Email | Primary email address |
| Why Priority | Text | One-line reason |
| Related Projects | Relation → Projects | |

**Body:** ongoing notes about the person, history, context.

### Reference database (LEGACY in v2 — read-only)

> **Status:** Legacy. v2 absorbed Reference into Supporting Documents (Type = `artifact`, Tag = `reference`). Read-only as above.

Mirrors `Reference/` folder.

| Property | Type | Purpose |
|----------|------|---------|
| Name | Title | Topic |

**Body:** evergreen reference material — SOPs, playbooks, saved links.

---

## Archive strategy

The file-based vault has an `Archive/` folder. In Notion, "archive" means two different things — keep them straight:

1. **Archive-the-status** (primary mechanism) — a `Status` property on each database whose values include `archived`. Monthly cleanup sets old records' Status to `archived`. Default views filter it out.
2. **Archive-the-page** (optional, for humans) — a sibling page under Chief of Staff called `📁 Archive Views` that just hosts linked-database views (one per database, each filtered to `Status = archived`). This is a navigational convenience, not a storage location. Records don't move there — they stay in their database and just flip Status.

Snapshots are a **third, separate** thing — they live in the `Snapshots` subpage (see "Snapshot destination" above) and have nothing to do with the Archive Status value or the Archive Views page.

- Every database has a `Status` property (see tables above).
- Default views on each database filter `Status != archived`.
- Monthly cleanup sets old records' Status to `archived`; nothing is moved or deleted.
- The Archive Views page is optional — create it during S4 only if the user wants a one-click view of everything archived.

---

## Linking records (relations replace wikilinks)

The file-based schema uses `[[wikilink]]` syntax. In Notion, linking between records is done via the **Relation** property defined on each database.

**Examples of required links (all implemented via Relation properties, not inline text):**

- **The file cabinet chain (four levels):** Domain ← Department ← Project ← Task. Each level links up to its parent via a Relation property. A task inherits its department and domain through its project.
- Every Briefing record has a Relation to the Projects it touched and the People referenced.
- Every Research / Draft / Action Plan has a Relation to the Briefing that spawned it.
- Tasks can link to the Briefing that spawned them (optional).
- The State Dashboard page uses embedded linked-database views to show Active Projects, High-Priority People, and Tasks due today or overdue.
- The Live Feed page references specific Briefings or Drafts by mentioning them with the `@` operator (type `@briefing-name` and Notion autocompletes a clickable link).

**Inline `@mentions`** are the Notion equivalent of in-body `[[wikilinks]]`. Use these in free-text sections (like State Dashboard's Watch List) where creating a formal Relation is overkill.

---

## Naming

- Briefings: `Briefing — YYYY-MM-DD` (Name property) + `Date` set to that date. The date in the Name is for quick scannability; Notion sorts by the Date property.
- Weekly Reviews: `Weekly Review — YYYY-MM-DD`
- Monthly Cleanups: `Monthly Cleanup — YYYY-MM`
- Research / Drafts / Action Plans: short topic or recipient name. Date goes in the `Date` property.
- Projects: short memorable name (same as file-based `project-slug`, but with spaces and caps allowed).
- People: `First Last`. No hyphen needed — Notion isn't filename-constrained.

---

## Snapshot vs. backup

See `references/vault-safety.md` for the distinction and full mechanics. In Notion, a snapshot is a duplicated page in the `Snapshots` subpage; a backup is a weekly export of the Chief of Staff parent page to encrypted off-Notion storage (`backup-setup.md` Path E).

---

## What Notion does NOT encrypt end-to-end

Notion encrypts your data in transit (HTTPS) and at rest (on their servers), but Notion's staff **can** technically read your workspace content. This is different from Obsidian + Cryptomator, where only you can read the files.

**Implication:** if the user's content is sensitive (private client details, health data, financial secrets, personal journal material), Notion may be a poor fit even with the weekly encrypted backup. The platform-choice flow steers sensitive-content users to Obsidian for this reason. If the user picked Notion despite sensitive content, the Chief of Staff should flag this during setup: "Your content sounds sensitive — Notion's live copy isn't end-to-end encrypted. Want to reconsider the platform?"

---

## Templates

Notion supports page templates — reusable structures that pre-fill the body of a new record. Create these inside each database during S4:

- **Domain template** — blank body with a placeholder: "Describe what this domain covers."
- **Department template** — blank body with a placeholder: "Describe what this department handles."
- **Project template** — pre-fills headings: Overview, Current Status, Key Decisions, Next Steps.
- **Task template** — blank body with a checklist block placeholder for sub-steps.
- **Briefing template** — pre-fills headings: Yesterday's Carryover, Today's Top 3, Schedule Snapshot, Messages That Need You, Watch List Check, Risks / Blockers, Recommended Actions.
- **Research template** — Question, Findings, Recommendation.
- **Action Plan template** — Goal, Steps, Timeline, Dependencies / Risks.
- **Draft template** — blank body with Subject / To properties pre-filled.

---

## Interaction with existing workspace

If the user already has a Notion workspace with other content, the Chief of Staff parent page sits alongside their existing content. It doesn't touch anything else. Shared databases (e.g., a People database the user already maintains) can be linked via Relation from the Chief of Staff's databases — ask the user during setup if any such databases exist, and offer to link rather than duplicate.

If the user has an existing life-journal page/database, the Chief of Staff reads it for context but doesn't own or modify it. Same rule as the file-based schema.
