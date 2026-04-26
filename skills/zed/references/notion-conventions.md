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
├── State Dashboard                  ← page (has page-level properties: Last Session At, setup_status)
├── Live Feed                        ← page
├── Snapshots                        ← page (holds state-snapshot-* page duplicates)
├── 🌐 Domains                       ← database (top of hierarchy: life entities — e.g., Business A, Personal)
├── 🏢 Departments                   ← database (functional areas within a domain — e.g., Marketing, Health)
├── 🚀 Projects                      ← database (specific initiatives within a department)
├── ✅ Tasks                          ← database (actionable to-dos within a project)
├── 📚 Briefings                     ← database
├── 🔍 Research                      ← database
├── ✉️ Drafts                         ← database
├── 📋 Action Plans                  ← database
├── 👤 People                        ← database
├── 📎 Reference                     ← database
└── 📁 Archive Views (optional)      ← page of linked views showing `Status = archived` across databases
```

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
- **## My Setup** → a simple two-column table (or a set of inline mentions). Fields: Name, Email, Role, Company/Team, Chief Name, **Personality** (one of: `Professional`, `Playful & lighthearted`, `Dry wit`, `Warm & encouraging`, or a custom description; defaults to `Professional` if blank), Alert Threshold, **Platform** (always `notion` for this schema).
- **## Active Projects** → an embedded linked view of the Projects database, filtered to `Status = active`, grouped by Department.
- **## Tasks Due** → an embedded linked view of the Tasks database, filtered to `Status != done AND Status != cancelled`, sorted by Due Date ascending. Shows what's on the plate right now.
- **## High-Priority People** → an embedded linked view of the People database, filtered to `Priority = high`.
- **## Watch List** → a bulleted list inside a toggle block. Each bullet is a short item. (A database is overkill here.)
- **## Missed Messages** → bulleted list inside a toggle.
- **## Open Decisions** → bulleted list inside a toggle.
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

### Research database

Mirrors `Research/` folder.

| Property | Type | Purpose |
|----------|------|---------|
| Name | Title | Topic (e.g., "Competitor Pricing") |
| Date | Date | When researched |
| Requested By | Relation → Briefings | The briefing that kicked off this research |
| Related Projects | Relation → Projects | |
| Status | Select | `active` / `archived` |

**Body:** Question, Findings, Recommendation.

### Drafts database

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

### Action Plans database

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
| Status | Select | `to-do` / `in-progress` / `blocked` / `done` / `cancelled` |
| Priority | Select | `urgent` / `high` / `medium` / `low` |
| Due Date | Date | When it's due (optional — not every task has a deadline) |
| Project | Relation → Projects | Which project this belongs to |
| Goal | Text | Optional — what outcome this task serves (for alignment without adding a 5th hierarchy level) |
| Notes | Text | Context, details, links |
| Waiting On | Text | Who or what is blocking this (only relevant when Status = blocked) |
| Related Briefing | Relation → Briefings | The briefing that spawned this task (if applicable) |

**Body:** Optional — checklists, sub-steps, or detailed notes. Use Notion's built-in checklist blocks for sub-dividing complex tasks rather than creating a 5th hierarchy level.

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

### Reference database

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
