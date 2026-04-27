# Flow: Version Migration

**What this flow does:** Walks the user through a version migration when SKILL.md's `Current:` version is ahead of `State Dashboard → My Setup → last_seen_version`. Also handles cross-plugin detection (older `ai-chief-of-staff` plugin still installed alongside Zed).

**When to load it:** Auto-loaded by Bootstrap step 3 when `last_seen_version` is missing or behind `Current:`. Also loaded by Bootstrap step 7 (cross-plugin detection branch). Can also be invoked by the user ("run the v2 migration").

**Prerequisites:** Bootstrap completed through step 2 (identity loaded). Universal Gates G1–G9 apply.

---

## Step 1 — Identify the migration

Read `Current:` from SKILL.md Skill Version section. Read `last_seen_version` from `State Dashboard → My Setup`.

- If `last_seen_version` is missing → user is pre-v2 (or new install). Treat as `v1`.
- If `last_seen_version < Current:` → run the migration matching the version delta.

This file currently knows two migrations:
- **v1 → v2** (Step 2 below)
- **v2 → v3** (Step 5 below — Captain's Log retirement)

If `last_seen_version = v1` AND `Current = v3`: run v1→v2 first, then v2→v3 (in that order, in the same session if possible). Future migrations append new sections.

---

## Step 2 — v1 → v2 migration offer

Tell the user (snapshot State Dashboard first per G2):

> "I see you're on Zed v1. v2 brings several changes:
>
> - **Supporting Documents** — a new database that consolidates Research, Action Plans, Reference, and Drafts into one place with Type tags.
> - **Reminders** — a new database for date-scoped nudges (different from Tasks, which are work to do).
> - **Surfaced Items** — a way to mark briefing items as suppress, handled, or remind-me-later.
> - **Time Sensitivity** — a new 0–5 property on Tasks for intuitive urgency rating.
>
> First question — how do you want to handle existing entries in your old Research / Action Plans / Reference / Drafts databases?
>
>   (a) **Move them all** — I'll infer Type from content. Anything I can't classify confidently lands as Type = `artifact`, Tag = `legacy-uncategorized`, and I'll surface a list at the end so you can adjust.
>   (b) **Leave them in place** — they stay readable, no new entries created in the old DBs. I'll re-ask once a quarter in your Monthly Cleanup, unless you say 'never ask again.'
>   (c) **Walk through one DB at a time** — I'll show you each old entry and ask where it goes."

Wait for the user's pick.

---

## Step 3 — Apply the user's choice

### Choice (a) — Move all

1. For each entry in the legacy DBs:
   - Apply the Type decision tree from `references/documentation-routing.md`.
   - If a clean match: create Supporting Document with inferred Type.
   - If no clean match: create with `Type = artifact, Tag = legacy-uncategorized`.
   - Track which entries fell through to `legacy-uncategorized`.
2. After all entries migrated, tell the user: "Migrated [N] entries. [M] of them I wasn't sure about — they're tagged `legacy-uncategorized`. Want to walk through them now or later?"
3. If "now" → load `flows/recall.md` filtered to `Tag = legacy-uncategorized` and let the user adjust each.
4. If "later" → leave them tagged; surfaces in next Monthly Cleanup.
5. Set `migration_v2_choice = moved`. Snapshot first per G2. Update `last_seen_version = v2`.

### Choice (b) — Leave in place

1. Do nothing to legacy DB content.
2. Set `migration_v2_choice = leave-in-place`. Snapshot first per G2. Update `last_seen_version = v2`.
3. Tell the user: "Got it. Your old entries stay where they are. New entries always go to Supporting Documents from now on. I'll check in once a quarter to see if you want to revisit — unless you tell me to stop asking."

### Choice (c) — Walk through

1. Set `migration_v2_choice = walking`. Snapshot first per G2.
2. For each legacy DB, surface each entry one at a time:
   - "[entry name] — Supporting Document with Type [inferred], leave it where it is, or skip?"
   - Apply user's choice per entry.
3. When all legacy entries handled, set `migration_v2_choice = complete`. Snapshot first. Update `last_seen_version = v2`.

### Quarterly re-prompt for choice (b) — handled by Bootstrap

If `migration_v2_choice = leave-in-place`, the Monthly Cleanup watcher (third run of every quarter — i.e., months 3 / 6 / 9 / 12) prompts:

> "Quarterly check — your legacy entries from the v2 migration are still in place. Revisit?"
>
> Options: `yes` / `not now` / `never ask again`

- `yes` → re-load this flow and re-run from Step 2.
- `not now` → leave state alone; quarterly re-prompt fires again next quarter.
- `never ask again` → set `migration_v2_choice = leave-permanent`. Quarterly re-prompt stops permanently.

**Deconflict (G9 conflict-detection).** When the quarterly re-prompt fires in a session, suppress the per-session legacy nudge from `references/documentation-routing.md` § Legacy DB read rule. At most one legacy-themed prompt per session. Tracked in session memory.

---

## Step 4 — Auto-complete on zero (handled by Bootstrap step 9)

When the legacy DB count hits zero (all entries migrated or removed) AND `migration_v2_choice` ∈ {`leave-in-place`, `walking`}: Bootstrap step 9 (Axiom 2 + legacy count) automatically flips `migration_v2_choice → complete`, snapshots first, and tells the user once: "All legacy entries have been migrated or removed. Marking the v2 migration complete."

---

## Step 5 — v2 → v3 migration offer (Captain's Log retirement)

**When this fires:** `last_seen_version = v2` AND `Current = v3` (or higher). Loaded automatically by Bootstrap step 3.

Tell the user (snapshot State Dashboard first per G2):

> "I see you're on Zed v2. v3 retires Captain's Log and replaces it with a new Personal Journal feature:
>
> - **Personal Journal** — a single home for reflective content (mood, energy, wins, challenges, gratitude, free thoughts). Auto-creates on first use. Lighter and more sustainable than Captain's Log.
> - **End-of-brief Personal Check-in** — one predictable journaling prompt at the end of every brief, easy to skip, easy to engage. Replaces the in-brief 'Captain's Log moments' nudges.
> - **Documentation routing now always asks** — 'log this' / 'remember this' / 'document this' no longer fall back to Captain's Log silently. The Chief asks where it goes.
>
> First question — what do you want to do with your existing Captain's Log entries?
>
>   (a) **Move them all to Archive** — I'll move every entry to `Chief-of-Staff/Archive/Captain's Log/` (or, in Notion, an Archive subpage). Nothing is deleted; they're preserved as legacy reference and recall still queries them with `[legacy CL]` annotation.
>   (b) **Leave them in place** — they stay readable in their original location, recall still queries them with `[legacy CL]` annotation. I'll re-ask once a quarter in your Monthly Cleanup, unless you say 'never ask again.'
>   (c) **Walk through one entry at a time** — I'll show you each old entry and ask whether to archive it, leave it, or skip."

Wait for the user's pick.

### Choice (a) — Move all to Archive

1. **Notion:** for each entry in the `Captain's Log` database, move the page to a new `Captain's Log Archive` page nested under the workspace's existing Archive structure (or create one). Use `notion-move-pages`. Don't change the entry contents.
2. **File-based** (`obsidian` / `markdown-folder` / `logseq`): create `Chief-of-Staff/Archive/Captain's Log/` if it doesn't exist. Move every file from `Chief-of-Staff/Captain's Log/` into the Archive folder. Preserve filenames and frontmatter.
3. After all entries archived, tell the user: "Archived [N] Captain's Log entries to [location]. Recall still finds them — they'll just show up tagged `[legacy CL]`."
4. Set `migration_v3_choice = moved`. Snapshot first per G2. Update `last_seen_version = v3`.

### Choice (b) — Leave in place

1. Do nothing to Captain's Log content.
2. Set `migration_v3_choice = leave-in-place`. Snapshot first per G2. Update `last_seen_version = v3`.
3. Tell the user: "Got it. Your Captain's Log entries stay where they are. New reflective content always goes to Personal Journal from now on. I'll check in once a quarter to see if you want to revisit — unless you tell me to stop asking."

### Choice (c) — Walk through

1. Set `migration_v3_choice = walking`. Snapshot first per G2.
2. Surface each Captain's Log entry one at a time:
   > "[Entry title] ([Date], [Type]) — archive, leave it, or skip?"
   - **archive** → move per Choice (a) mechanics for this single entry.
   - **leave** → no change.
   - **skip** → no change, don't ask again this session.
3. When all entries handled, set `migration_v3_choice = complete`. Snapshot first. Update `last_seen_version = v3`.

### Quarterly re-prompt for Choice (b) — handled by Monthly Cleanup

If `migration_v3_choice = leave-in-place`, the Monthly Cleanup watcher (third run of every quarter — months 3 / 6 / 9 / 12) prompts:

> "Quarterly check — your Captain's Log entries from the v3 migration are still in place. Revisit?"
>
> Options: `yes` / `not now` / `never ask again`

- `yes` → re-load this flow and re-run from Step 5.
- `not now` → leave state alone; quarterly re-prompt fires again next quarter.
- `never ask again` → set `migration_v3_choice = leave-permanent`. Quarterly re-prompt stops permanently.

**Deconflict (G9 conflict-detection).** When the v3 quarterly re-prompt fires in a session, suppress any other legacy-themed prompts (v2 legacy nudges, cross-plugin nudge). At most one legacy-themed prompt per session. Tracked in session memory.

### Auto-complete on zero (handled by Bootstrap step 9)

When the Captain's Log entry count hits zero (all entries migrated or removed) AND `migration_v3_choice` ∈ {`leave-in-place`, `walking`}: Bootstrap step 9 automatically flips `migration_v3_choice → complete`, snapshots first, and tells the user once: "All Captain's Log entries have been archived or removed. Marking the v3 migration complete."

### Personal Journal stays lazy

The v3 migration does NOT proactively create the Personal Journal during the migration. It's still auto-created on first use per `flows/journal.md` § Step 1. Migration is about handling old Captain's Log entries, not setting up the new structure.

---

## § Cross-Plugin Detection

**Loaded by Bootstrap step 7 when:** the older `ai-chief-of-staff:log` skill is detected in the available skills list AND `legacy_plugin_choice` is missing or `dismissed`.

### Why this matters

The older `ai-chief-of-staff` plugin contains its own standalone `log` skill that routes "log this" cold-starts through different logic than Zed's documentation-routing. If both plugins are installed, cold-start `log this` (said before Zed has session context) may bypass Supporting Documents entirely.

### The one-time prompt

Snapshot State Dashboard first per G2, then surface (once per session, deconflicted with the v2 migration prompt — at most one legacy-themed prompt per session):

> "I see both the older `ai-chief-of-staff` plugin and the newer `zed-chief-of-staff` plugin are installed. Documentation routing now lives in Zed — keeping both can cause 'log this' said cold (before this session has context) to route through the older skill, bypassing Supporting Documents.
>
> Want me to walk you through uninstalling the older plugin? (yes / not now / never ask again)
>
> Until you uninstall, prefer saying 'log this' inside an active Zed session rather than as a cold start."

### Apply the user's choice

- **yes** → walk the user through the uninstall. Steps depend on how plugins are installed (Cowork plugin manager, or manual). Set `legacy_plugin_choice = uninstalled` after confirmation. Snapshot first.
- **not now** → set `legacy_plugin_choice = dismissed`. Re-prompt next session.
- **never ask again** → set `legacy_plugin_choice = permanent`. Stop prompting.

### Known limitation

If user picked `dismissed`, cold-start `log this` (said before Zed gets session context) may still route through the older plugin's logic for the duration of the deferral. This is bounded by user choice — re-prompted next session. Recommendation in the prompt copy above tells the user how to work around it manually.

---

## Snapshot rule

Every State Dashboard write in this flow (setting `migration_v2_choice`, `migration_v3_choice`, `last_seen_version`, `legacy_plugin_choice`) takes a snapshot first per G2. If the snapshot fails: STOP, do not write, surface the failure, ask how to proceed.

Migration of legacy DB entries to Supporting Documents does NOT touch State Dashboard — no snapshot needed for those operations themselves. Same for v3 archival of Captain's Log entries — moving files / Notion pages does not touch State Dashboard. Only the choice-tracking writes hit State Dashboard.
