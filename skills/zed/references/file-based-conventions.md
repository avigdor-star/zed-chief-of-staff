# File-Based Vault Conventions for Chief of Staff Documents

> **Applies to:** Obsidian, Logseq, and plain markdown folders. All three store your notes as regular files on your computer, so the rules below are identical across them.
>
> **Using Notion instead?** Stop here and load `references/notion-conventions.md` — Notion uses databases and properties instead of folders and frontmatter, so its schema is different.

All documents created by the Chief of Staff system live in the user's vault and must follow
these conventions to stay linked, searchable, and well-organized.

## Vault Location

All Chief of Staff documents live under: `Chief-of-Staff/` (inside the user's vault or workspace)

```
Chief-of-Staff/
├── _index.md                   ← One-screen map of the vault. Read FIRST every session.
├── State Dashboard.md          ← Living status dashboard (the brain)
├── Live Feed.md                ← Running intake file
├── Domains/                    ← One file per life domain (top of hierarchy)
├── Departments/                ← One file per department (functional areas within a domain)
├── Projects/                   ← One subfolder per active long-lived project
├── Tasks/                      ← One file per task (actionable to-dos within a project)
├── Briefings/                  ← Daily briefings, weekly reviews
├── Research/                   ← Research outputs from sub-agents
├── Drafts/                     ← Email drafts, message drafts
├── Action-Plans/               ← Plans, breakdowns, decision docs
├── People/                     ← One file per high-priority person
├── Reference/                  ← Evergreen reference material — SOPs, playbooks, saved links
└── Archive/                    ← Old briefings, state snapshots, completed items
```

**Snapshots vs. backups:** see `references/vault-safety.md` for the distinction, snapshot mechanics, and restoration. In short: snapshots live inside this vault for quick rollback; backups live off-machine for survival.

**Journal placement:** Personal journal entries do NOT live under `Chief-of-Staff/`. If the user keeps a journal, it lives elsewhere in their vault. The Chief of Staff reads it (if present) but does not own or create it.

## Frontmatter

Every document created by the system must have YAML frontmatter. This makes them searchable
and filterable in any file-based app (Obsidian, Logseq) and keeps structured data in the file
itself for plain-markdown setups.

### State Dashboard frontmatter (Obsidian / plain markdown folder):

In addition to the existing `type`, `last_updated`, and `tags` fields, the State Dashboard frontmatter includes the `rollout_reminder` block used by the Pacer pattern. See `references/rollout-reminder.md` for field semantics.

```yaml
---
type: state-dashboard
last_updated: YYYY-MM-DD
platform: obsidian          # or logseq / markdown-folder
chief_name: ""              # user's chosen name for the Chief
personality: Professional   # Professional / Playful & lighthearted / Dry wit / Warm & encouraging / custom
setup_status: complete      # in_progress / complete / complete-with-warning
rollout_reminder:
  last_mentioned: YYYY-MM-DD
  last_contextual_mention: YYYY-MM-DD   # or leave blank / null
  next_eligible: YYYY-MM-DD
  last_checkin_response: overwhelming   # or just_right / ready_for_more / null
  last_action: null                     # or snooze / express
  express_partial: false
  express_installed: []
---
```

### State Dashboard properties (Logseq):

Logseq flattens nested YAML, so the `rollout_reminder` block is expressed as hyphenated `::` page properties at the top of the State Dashboard page:

```
rollout_reminder-last-mentioned:: YYYY-MM-DD
rollout_reminder-last-contextual-mention:: YYYY-MM-DD
rollout_reminder-next-eligible:: YYYY-MM-DD
rollout_reminder-last-checkin-response:: overwhelming
rollout_reminder-last-action:: (blank, or snooze / express)
rollout_reminder-express-partial:: false
rollout_reminder-express-installed:: (comma-separated list, or blank)
```

Multi-value fields (`express_installed`) are comma-separated, matching the existing Logseq convention.

### Briefing frontmatter:
```yaml
---
type: briefing
date: YYYY-MM-DD
sources: [email, calendar, tasks]
---
```

### Weekly review frontmatter:
```yaml
---
type: weekly-review
date: YYYY-MM-DD
week: YYYY-WNN
---
```

### Research document frontmatter:
```yaml
---
type: research
date: YYYY-MM-DD
topic: [topic description]
requested-by: [briefing/interactive session]
---
```

### Draft frontmatter:
```yaml
---
type: draft
date: YYYY-MM-DD
for: [recipient name]
regarding: [subject]
---
```

### Action plan frontmatter:
```yaml
---
type: action-plan
date: YYYY-MM-DD
project: [project name]
---
```

### Domain file frontmatter (lives in `Domains/`):
```yaml
---
type: domain
description: [one-line summary of what this domain covers]
status: [active | archived]
---
```

### Department file frontmatter (lives in `Departments/`):
```yaml
---
type: department
domain: "[[Domains/domain-name]]"
description: [what this department covers]
status: [active | archived]
---
```

### Project file frontmatter (lives in `Projects/[project-name]/`):
```yaml
---
type: project
department: "[[Departments/department-name]]"
status: [active | paused | completed]
started: YYYY-MM-DD
---
```

### Task file frontmatter (lives in `Tasks/`):
```yaml
---
type: task
status: [to-do | in-progress | blocked | done | cancelled]
priority: [urgent | high | medium | low]
due: YYYY-MM-DD
project: "[[Projects/project-name]]"
goal: [optional — what outcome this task serves]
waiting-on: [who or what is blocking this, if status = blocked]
---
```

### Person file frontmatter (lives in `People/`):
```yaml
---
type: person
relationship: [client | collaborator | vendor | family | other]
priority: [high | medium | low]
---
```

### Reference file frontmatter (lives in `Reference/`):
```yaml
---
type: reference
topic: [topic]
---
```

## Wikilinks

Link documents to each other using `[[wikilink]]` syntax. Obsidian and Logseq both render
these as clickable links and provide a visual map of how notes connect to each other. In a
plain markdown folder the links appear as plain text — still useful for searching and
manually following references, even if not clickable.

### Required links:

**The file cabinet chain (four levels):** Each level links up to its parent via a wikilink in its frontmatter. A task links to its project, a project links to its department, a department links to its domain. This chain lets you trace any task back to its life domain.

**In every briefing:**
- Link to the State Dashboard: `[[State Dashboard]]`
- Link to any research/drafts/action plans created during the session
- Link to the previous briefing: `[[Briefings/YYYY-MM-DD-briefing|Yesterday's Briefing]]`

**In every research doc / draft / action plan:**
- Link back to the briefing or session that requested it
- Link to the relevant project (if the user has project files elsewhere in their vault)
- Link to related documents if they exist

**In the State Dashboard:**
- Link to the latest briefing
- Link to active action plans
- Link to relevant project files
- Link to tasks that are due today or overdue

**In the Live Feed:**
- Link to related documents when clearing items (e.g., "Handled — see [[Drafts/YYYY-MM-DD-topic|draft]]")

### Link format examples:

```markdown
See [[State Dashboard]] for current project status.
Created research doc: [[Research/YYYY-MM-DD-competitor-pricing|Competitor Pricing Research]]
Follow-up from [[Briefings/YYYY-MM-DD-briefing|yesterday's briefing]].
```

Use the `[[path|display text]]` format when the file path is long — it keeps the document readable.

## File Naming

- Briefings: `YYYY-MM-DD-briefing.md`
- Weekly reviews: `YYYY-MM-DD-weekly-review.md`
- Monthly cleanups: `YYYY-MM-monthly-cleanup.md`
- Research: `YYYY-MM-DD-[topic-slug].md`
- Drafts: `YYYY-MM-DD-[recipient-or-topic].md`
- Action plans: `YYYY-MM-DD-[topic-slug].md`
- State snapshots (in-vault): `state-snapshot-YYYY-MM-DD-HHMM.md` (append `-2`, `-3`, … if the exact timestamp already exists)
- Feed archives: `feed-archive-YYYY-MM-DD.md`
- Domain files: `Domains/[domain-slug].md` (e.g., `Domains/my-agency.md`, `Domains/personal.md`)
- Department files: `Departments/[department-slug].md` (e.g., `Departments/marketing.md`, `Departments/health.md`)
- Project folders: `Projects/[project-slug]/` (lowercase, hyphens, short)
- Task files: `Tasks/[task-slug].md` (e.g., `Tasks/reply-to-vendor-quote.md`)
- Person files: `People/First-Last.md`
- Reference files: `Reference/[topic-slug].md`

## Templates

### Briefing Template

```markdown
---
type: briefing
date: YYYY-MM-DD
sources: [email, calendar, tasks]
---

# Briefing — YYYY-MM-DD

> Related: [[State Dashboard]] | [[Briefings/YYYY-MM-DD-briefing|Previous Briefing]]

## Yesterday's Carryover
-

## Today's Top 3
1.
2.
3.

## Schedule Snapshot
-

## Messages That Need You
-

## Watch List Check
-

## Risks / Blockers
-

## Recommended Actions
-

---
*Generated by Chief of Staff · [[State Dashboard]]*
```

### Research Template

```markdown
---
type: research
date: YYYY-MM-DD
topic: [topic]
requested-by: [briefing/interactive session]
tags: [cos, research]
---

# Research: [Topic]

> Requested during [[Briefings/YYYY-MM-DD-briefing|today's briefing]]
> Related project: [[link to project if applicable]]

## Question
[What we're trying to answer]

## Findings
[Research results]

## Recommendation
[What to do with this information]

---
*Generated by Chief of Staff sub-agent*
```

### Action Plan Template

```markdown
---
type: action-plan
date: YYYY-MM-DD
project: [project name]
---

# Action Plan: [Title]

> Related: [[State Dashboard]] | [[link to project]]

## Goal
[What success looks like]

## Steps
1.
2.
3.

## Timeline
-

## Dependencies / Risks
-

---
*Generated by Chief of Staff sub-agent*
```

### _index.md template

Written as the FIRST file during S4 (setup), and regenerated on demand if the Step 0 "missing _index.md fallback" triggers. This is the vault's contract file — every session reads it first.

```markdown
---
type: index
---

# Chief of Staff Vault — Index

> Read this first. One-screen map of where everything lives, what it's named, and how it's tagged.
> When you create a new document in this vault, this file tells you where it belongs.

> **Platform note:** This index describes the file-based vault contract (Obsidian, Logseq, or plain markdown folder). If the user picked Notion during setup, see the skill's `references/notion-conventions.md` for the equivalent schema (databases instead of folders, properties instead of frontmatter).

---

## Folders

| Folder | Purpose | Filename pattern | Lifespan |
|--------|---------|------------------|----------|
| `Domains/` | One file per life domain (top of hierarchy) | `[domain-slug].md` | Ongoing |
| `Departments/` | One file per department (functional areas within a domain) | `[department-slug].md` | Ongoing |
| `Projects/` | One subfolder per active long-lived project | `Projects/[project-name]/[notes].md` | Until project ends → `Archive/` |
| `Tasks/` | One file per actionable to-do (linked to a project) | `[task-slug].md` | Until done or cancelled → `Archive/` |
| `Briefings/` | Daily briefings and weekly/monthly reviews | `YYYY-MM-DD-briefing.md`, `YYYY-MM-DD-weekly-review.md`, `YYYY-MM-monthly-cleanup.md` | Move to `Archive/` after 30 days |
| `Research/` | Research outputs from sub-agents | `YYYY-MM-DD-[topic-slug].md` | Keep while relevant, then archive |
| `Drafts/` | Email and message drafts awaiting send | `YYYY-MM-DD-[recipient-or-topic].md` | Delete once sent |
| `Action-Plans/` | Plans, breakdowns, decision docs | `YYYY-MM-DD-[topic-slug].md` | Keep for the life of the project |
| `People/` | One file per high-priority person | `[First-Last].md` | Ongoing |
| `Reference/` | Evergreen reference material | `[topic-slug].md` | Ongoing |
| `Archive/` | State Dashboard snapshots, old briefings, completed items | `state-snapshot-YYYY-MM-DD-HHMM.md`, `feed-archive-YYYY-MM-DD.md` | Keep indefinitely |

## Root-level files

| File | Purpose | Who writes to it |
|------|---------|------------------|
| `State Dashboard.md` | Living status file. Single source of truth. | Chief of Staff (primary). Weekly Review writes Trends on Fridays. User may add content manually. Watchers do NOT write here. Every write must snapshot first (hard gate). |
| `Live Feed.md` | Running intake for watchers and messages | Watchers append (ALERTS / INFO / NOTICED sections). Chief of Staff clears handled items. |
| `_index.md` | This file | Humans only, on purpose |

## Journal placement

Personal journal entries do **not** live in `Chief-of-Staff/`. If the user keeps a journal, it lives elsewhere in the vault. The Chief of Staff reads it (if present) but does not own it.

## Naming rules

- Dates: `YYYY-MM-DD` format, always.
- Topic slugs: lowercase, hyphens, no spaces.
- People files: `First-Last.md`.
- Project folders: short, memorable slug.

## Frontmatter rules

Every document must start with YAML frontmatter. Full templates live in the skill's `references/file-based-conventions.md`.

## Wikilinks

- Every new document must link back to `[[State Dashboard]]`.
- Briefings link to the previous briefing.
- Research/drafts/action-plans link to the briefing that requested them.

## Backup

This vault must be backed up to an end-to-end encrypted location. See the skill's `references/backup-setup.md`. Backup status is tracked in `State Dashboard → System Health → Connectors → Backup`.

## Snapshots vs. backups

- **Snapshot** = a copy inside this vault (e.g., `Archive/state-snapshot-YYYY-MM-DD-HHMM.md`). Used for rollback within the vault.
- **Backup** = the whole vault, mirrored to an encrypted off-machine location. Not the same thing. Snapshots are useful; backups are survival.

**Restoring from a snapshot:** ask the Chief of Staff, "Restore the State Dashboard from the latest snapshot." See the skill's `references/file-based-conventions.md` → "Snapshot restoration" for the procedure.

---

*This index is the contract. If a new document doesn't fit any row in the Folders table, either the document is in the wrong place or this index needs updating — not the other way around.*
```

## Snapshot Rules (in-vault rollback copies)

Snapshot mechanics, the snapshot-first hard gate, and restoration procedures all live in `references/vault-safety.md`. Two file-based-only specifics that don't fit there:

- If the Live Feed exceeds 200 lines, move all HANDLED entries to `Archive/feed-archive-YYYY-MM-DD.md`.
- Monthly cleanup moves briefings older than 30 days to `Archive/`.

## Backup Rules (off-machine encrypted survival)

- The whole `Chief-of-Staff/` vault must be backed up to an end-to-end encrypted location.
- See `references/backup-setup.md` for the four free options (Cryptomator, git-crypt, Proton Drive, Syncthing) and the AI-assisted setup walkthrough.
- The Chief of Staff verifies backup status during Step 1 of every session and surfaces it in `State Dashboard → System Health → Connectors → Backup`.
- If no backup is detected, the Chief of Staff alerts the user and offers to walk them through `backup-setup.md`.

## Platform Differences — Logseq

Logseq is block-based and uses different mechanics than Obsidian for properties, page placement, and tags. The frontmatter, folder layout, and tag schemas above are written in Obsidian's style. For Logseq users, use the mappings below — the overall schema is the same, just written in Logseq's syntax.

Everything not listed here (wikilinks `[[...]]`, snapshot rules, file naming conventions, backup rules, the snapshot-vs-backup distinction) is **unchanged** — those work identically in Logseq.

### Property syntax: YAML → `::`

Logseq doesn't use YAML frontmatter. It uses **page properties** written as `key:: value` on lines at the top of the page (before any other content).

Mapping every Obsidian frontmatter block to Logseq properties:

**Briefing:**
```
type:: briefing
date:: YYYY-MM-DD
sources:: email, calendar, tasks
```

**Weekly review:**
```
type:: weekly-review
date:: YYYY-MM-DD
week:: YYYY-WNN
```

**Research:**
```
type:: research
date:: YYYY-MM-DD
topic:: [topic]
requested-by:: [briefing/interactive session]
```

**Draft:**
```
type:: draft
date:: YYYY-MM-DD
for:: [recipient]
regarding:: [subject]
```

**Action plan:**
```
type:: action-plan
date:: YYYY-MM-DD
project:: [project name]
```

**Domain:**
```
type:: domain
description:: [one-line summary]
status:: active
```

**Department:**
```
type:: department
domain:: [[Domains/domain-name]]
description:: [what this department covers]
status:: active
```

**Task:**
```
type:: task
status:: to-do
priority:: medium
due:: YYYY-MM-DD
project:: [[Projects/project-name]]
goal:: [optional]
waiting-on:: [optional]
```

**Project / Person / Reference:** same field-for-field mapping — every YAML key becomes a `key:: value` line. Projects now include a `department:: [[Departments/department-name]]` property.

### Page placement: folders → namespaces (or folders)

Logseq supports two placement styles. Pick one and stay consistent.

**Style 1 — namespaces (Logseq-native).** Pages live in `pages/` with `___` as the namespace separator. A Briefing becomes:

```
pages/Chief-of-Staff___Briefings___YYYY-MM-DD-briefing.md
```

Logseq renders this in its sidebar as a nested tree (Chief-of-Staff → Briefings → YYYY-MM-DD-briefing). Same visual result as folders in Obsidian.

**Style 2 — plain folders.** If the user configured Logseq to respect subfolders, the Obsidian layout (`Chief-of-Staff/Briefings/YYYY-MM-DD-briefing.md`) works as-is.

**Journal exception.** Logseq has a built-in `journals/` folder for daily notes. If the user already uses journals, briefings can live there instead — but that's a user preference, not a Chief-of-Staff rule. Ask at setup if they want briefings in `journals/` or under `Chief-of-Staff/Briefings/`.

### Quick reference table

| Concept | Obsidian / plain markdown | Logseq |
|---------|---------------------------|--------|
| Frontmatter | YAML between `---` lines | `key:: value` lines at top |
| Folders | `Chief-of-Staff/Briefings/file.md` | `pages/Chief-of-Staff___Briefings___file.md` (namespace) or plain folder |
| Wikilinks | `[[file]]` | `[[file]]` (same) |
| Daily notes | `Briefings/YYYY-MM-DD-briefing.md` | `journals/YYYY_MM_DD.md` (optional) or same as Obsidian |
| Snapshots | `Archive/state-snapshot-YYYY-MM-DD.md` | same |
| File naming | `YYYY-MM-DD-[slug].md` | same |
| Backup | off-machine encrypted (see `backup-setup.md`) | same |

If any of these mappings conflict with how the user has already set up Logseq, defer to their setup and note it in the State Dashboard.

## Interaction with Existing Vault

The Chief of Staff folder is self-contained. It links OUT to other parts of the vault
(like project folders or journal files) but doesn't depend on them. If those files don't
exist, the system works fine without them.

If the user has an existing life-snapshot or journal, use the existing conventions for
those files — don't create a competing format.
