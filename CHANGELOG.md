# Zed Plugin — Changelog

Changes to the skill during beta development. Newest first.

---

## 2026-04-30

### Backup setup — Path E Option A rewritten for Cowork sandbox reality

Discovered during a real install that the previously documented Option A (single Cowork scheduled task) was architecturally broken: Cowork scheduled tasks run in an isolated sandbox and cannot write to `/Volumes/` or `~/Documents/`. Following the doc as written produced a "successful" weekly run that never actually placed an encrypted backup into the user's Cryptomator vault — exports landed silently in a session-internal outputs folder.

#### Files edited

- `skills/zed/references/backup-setup.md` — Rewrote Path E Option A. Added an architectural-reality warning at the top explaining the Cowork sandbox limitation (cannot write to `/Volumes/` or `~/Documents/`; writes land in `~/Library/Application Support/Claude/local-agent-mode-sessions/<id>/<id>/local_<id>/outputs/`). Split Option A into two required components: Component 1 (Cowork scheduled task that exports to its session outputs) and Component 2 (native Mac launchd mover that finds the export in the sessions tree and moves it into the unlocked Cryptomator vault). Updated the embedded scheduled-task prompt: removed the impossible `~/Documents/` write target and the "ENCRYPT" step that claimed to verify vault placement; replaced with `outputs/CoS-Notion-Backup/YYYY-MM-DD/` and a new Status enum value `EXPORTED_PENDING_ENCRYPTION` that is flipped to `VERIFIED` by the mover. Added a Component 2 walkthrough describing the mover's seven-step behavior, the two files it generates (`cos-backup-mover.sh` + `com.<owner>.cos-backup-mover.plist`), the install steps (`~/scripts/`, `~/Library/LaunchAgents/`, `launchctl load`), the test procedure, and the default schedule (Cowork at Tuesday 12:00, mover at Tuesday 12:30). Updated the parallel calendar reminder to reference the +30 mover time. Added a "Pre-approve connector permissions" step prompting the user to "Run now" once after task creation so future runs don't pause mid-task on first-time permission grants.

#### What did NOT change (deliberately)

- Option B (manual export). It runs entirely on the user's Mac with no Cowork sandbox involvement; the existing instructions are correct.
- Part 2 (encryption section) and the per-path Verify procedures. Option A's encryption is now handled by Component 2; Part 2's instructions still apply to Option B.
- All other reference files. The bug was localized to Path E Option A.

#### Discovery context

Real install on 2026-04-30 followed the original Option A. The Cowork scheduled task ran successfully on 2026-04-28 per `list_scheduled_tasks` (`lastRunAt: 2026-04-28T19:10:15.668Z`), but a `find` of the user's home folder revealed the export had landed at `~/Library/Application Support/Claude/local-agent-mode-sessions/<long-path>/outputs/CoS-Notion-Backup/2026-04-28/`, never in the Cryptomator vault at `/Volumes/chief-of-staff-backup/`. Patched in this session by adding a Mac launchd mover (now documented as Component 2) and verifying the existing 2026-04-28 export moved into the encrypted vault.

#### G7' audit (this change)

- **Null-handling check.** New `EXPORTED_PENDING_ENCRYPTION` Status option on Backup row. Existing reads of Backup Status (Bootstrap step 8 / Axiom 1 staleness check) treat any non-VERIFIED value as still-pending; no special handling required.
- **Status filter audit.** Briefing rendering does not filter on Backup Status — it surfaces non-VERIFIED states regardless of the specific value. No filter updates needed.
- **Notion saved views audit.** State Dashboard's Backup row is a single record, not a database view filter. N/A.
- **External integration audit.** No watcher prompts referenced the previous Status enum directly. N/A.

#### G9 PRC review

- **Provenance.** The Backup row's Status / Last Exported / Last Verified fields all carry the writer (Cowork task vs. Mac mover) in the Method/Note text. Mover writes a `.last-mover-run` timestamp file inside the vault as a parallel provenance signal that survives independent of the State Dashboard.
- **Reversal.** All Backup row writes are State-Dashboard writes subject to G2 snapshots (existing). Mover moves are not destructive (mv preserves data; vault is the single source of truth for encrypted copies). User can re-run the mover manually at any time.
- **Conflict-Detection.** Mover skips dated subfolders already present in the vault (no overwrites). Cowork task and mover are decoupled by file location and timing — neither blocks on the other.

---

## 2026-04-27

### Zed v3 — Personal Journal feature (Captain's Log retired)

Mikoshi-validated plan (YELLOW → resolved by adopting full v3 release with v2-pattern migration). Added in response to user feedback that Captain's Log had become a burden; replaced with a lighter, more sustainable Personal Journal anchored by a single end-of-brief Personal Check-in.

#### Files created

- `skills/zed/references/personal-journal.md` — Personal Journal schema (Date / Mood / Energy / Wins / Challenges / Gratitude / Free text), per-platform storage (Notion / Obsidian / Logseq / plain markdown), auto-create-on-first-use behavior, no auto-generated first entry rule, recall pointer, graduation rule pointer, G9 PRC summary.
- `skills/zed/flows/journal.md` — Personal Journal engagement flow. Step 1 first-use auto-create gate. Step 2 four-pick routing (draft now / find calendar time / blank entry / suggest prompt based on the week). Step 3 per-pick mechanics. Step 4 engagement-skip tracking with soft-mute (5 skips → ask once to dial back to weekly or pause). Step 5 graduation offer (PJ Free Text → SD brainstorm). G9 PRC summary.

#### Files edited

- `skills/zed/SKILL.md` — bumped `Current:` from v2 to v3; added v3 changelog entry at top (parser-pinned format preserved). Outline gains a new row for Personal Journal phrases routed to `flows/journal.md`; documentation-routing row updated to drop Captain's Log as silent fallback; recall row updated to mention Personal Journal. Bootstrap step 2 identity load extended with `migration_v3_choice`, `personal_journal_location`, `journal_check_in_skips`, `journal_check_in_frequency`. Bootstrap step 9 gains v3 auto-complete-on-zero check (parallel to v2's). Stable Names: Personal Journal added (workspace top level, file-based subfolder); Captain's Log marked LEGACY but preserved so existing user data isn't orphaned. Notion / file-based / Logseq frontmatter sections add Personal Journal entries and mark Captain's Log entries legacy. State Dashboard property list adds the four new v3 fields. Glossary adds entries for Personal Journal, Personal Check-in, Graduation. Reference Manifest adds `personal-journal.md` and marks `captains-log.md` legacy; mentions `flows/journal.md`.
- `skills/zed/flows/briefing.md` — Step 1 source-scan: removed "Captain's Log (last 7 days)" entry from both file-based and Notion paths; added explicit note that Personal Journal is NOT scanned during briefings (recall via `flows/recall.md` if needed). Step 5 Proactive documentation: replaced "Captain's Log moments" prompt with a "Working-doc moments" prompt; added explicit clarifying note that personal/reflective content lives in the new dedicated end-of-brief Personal Check-in (Step 6) — never scattered through the brief. NEW Step 6 — Personal Check-in: render gate keyed on `journal_check_in_frequency`, personality-aware default wording, engages → load `flows/journal.md`, skips → increment counter, suppresses for migration sessions and alert auto-fires. Old Step 6 (Housekeeping) renumbered to Step 7.
- `skills/zed/flows/documentation-routing.md` — fully rewritten flow file. v3: ALWAYS asks where ambiguous content goes (no silent default). Personal Journal is reserved for explicit journal phrases (route through `flows/journal.md`, not here). Legacy "captain's log: …" phrase gracefully redirects to `flows/journal.md` with optional one-time tip after 3 redirects. Stripped Captain's Log graduation rule from this flow (lives in `references/documentation-routing.md` § graduation rule).
- `skills/zed/references/documentation-routing.md` — graduation rule reframed from "Captain's Log Idea ↔ SD brainstorm" to "Personal Journal Free Text ↔ SD brainstorm" with legacy fallback for users mid-migration. Type-transitions list updated. Snapshot-rule list updated to include all v3 State Dashboard fields requiring G2.
- `skills/zed/flows/recall.md` — search rules now include Personal Journal (full-text on body sections + Mood / Energy / Date filters); Captain's Log demoted to legacy (read-only, surfaced as `[legacy CL]`). Single-system query patterns table updated. Result format extended to handle Personal Journal entries (mood instead of Type). Graduation offers section now covers BOTH Personal Journal Free Text → SD brainstorm AND legacy Captain's Log Idea → SD brainstorm (at most one per recall per G9).
- `skills/zed/flows/setup.md` — S4.5 fully rewritten. Setup no longer creates Captain's Log. New users get told once that Personal Journal exists and will auto-create on first use. S5 sets `last_seen_version = v3`, `migration_v2_choice = complete`, `migration_v3_choice = complete`, `personal_journal_location = blank`, `journal_check_in_skips = 0`, `journal_check_in_frequency = every-brief` for new users.
- `skills/zed/flows/version-migration.md` — added Step 5: v2→v3 migration offer with three branches (move-all = archive Captain's Log to `Archive/Captain's Log/` or Notion Archive subpage; leave-in-place = quarterly re-prompt; walking = one entry at a time). Auto-complete-on-zero hooked into Bootstrap step 9. Quarterly re-prompt for choice (b) integrated into Monthly Cleanup. Snapshot rule extended to list `migration_v3_choice` among State Dashboard writes requiring G2.
- `skills/zed/references/notion-conventions.md` — Top-level structure tree gains Personal Journal database (workspace top level, NOT under CoS parent — same pattern Captain's Log used) and marks Captain's Log legacy. New `Personal Journal database (added v3)` section with property table and saved view recommendations. New `Captain's Log database (LEGACY in v3)` section preserving the schema for read access. State Dashboard property list at the top of the tree extended with the four new v3 fields.
- `skills/zed/references/file-based-conventions.md` — Vault Location tree gains `Personal Journal/` subfolder (auto-created on first use) and `Captain's Log/` legacy subfolder. Removed the v2 statement "personal journal entries do NOT live under Chief-of-Staff/" — that's wrong in v3. New "Personal Journal entry frontmatter" section with markdown-headers body convention. New "Captain's Log entry frontmatter (LEGACY in v3)" section preserving the schema. State Dashboard frontmatter extended with the four v3 fields; bumped `last_seen_version` example from v2 to v3.

#### Files deleted

- `skills/zed/flows/captains-log-recall.md` — the deprecation stub left over from v2 (was just a redirect notice). Safe to remove now per the v2 changelog note. `flows/recall.md` is the canonical cross-system recall flow.

#### Files NOT deleted (intentional)

- `skills/zed/references/captains-log.md` — KEPT as legacy documentation. The v3 plan archives existing Captain's Log entries (or leaves them in place per user choice), and recall.md still queries them. Schema documentation needs to remain accessible.

#### G7' audits required for v3 ship

- **Null-handling check.** Personal Journal entries handle null/missing optional fields gracefully (Mood / Energy / Wins / Challenges / Gratitude / Notes can all be empty). Recall queries on Mood use partial match to handle null cases. `journal_check_in_skips` defaults to 0 if missing. `journal_check_in_frequency` defaults to `every-brief` if missing.
- **Status filter audit.** No Status field on Personal Journal — N/A. Briefing source-scan no longer includes Captain's Log queries (removed). Recall correctly distinguishes Personal Journal from legacy Captain's Log.
- **Notion saved views audit.** Personal Journal database gets three default views on auto-create (Recent / By Mood / By Month). Captain's Log views unchanged (legacy DB).
- **External integration audit.** No watcher prompts referenced Captain's Log directly — N/A.

#### G9 PRC review for v3

- **Provenance.** Personal Journal entries record Date + cos_id + Mood (if supplied) + Energy (if supplied). All entries are user-authored — Chief never auto-writes. Engagement skips are tracked per session and persisted to State Dashboard with G2 snapshots.
- **Reversal.** Every state-changing verb has an inverse: entries are editable / deletable; calendar blocks (from Pick 2) are user-deletable; soft-mute decision is reversible by saying "turn journal check-in back on"; graduation is reversible (delete the SD; original PJ entry preserved).
- **Conflict-Detection.** Single conflict point is the graduation offer — Chief asks before promoting Free Text to a Supporting Document. Engagement skip tracking deconflicts the soft-mute prompt against other end-of-brief prompts. Migration prompts deconflict per existing G9 rule (at most one legacy-themed prompt per session).

#### Out of scope (intentional)

- Personal Journal recall by Mood with mood-similarity expansion (e.g., "show me when I felt down" matching "low," "tired," "exhausted") — deferred. Current implementation uses partial match only.
- Mood-trend charting in briefings — deferred. Personal Journal entries don't surface in briefings at all in v3.
- The standalone `obsidian-journal` skill (in a separate plugin) is NOT modified. Bootstrap step 7 already detects parallel installations of CoS-themed plugins; for users running both, the standalone skill takes precedence on Obsidian setups.

---

## 2026-04-26

### Zed v2 — 5-feature deployment

Mikoshi-validated plan (94/100, Path-decisively-into-Alignment). 5 iterations from 60 → 73 → 80 → 89 → 94. Plan files saved at session outputs.

#### Files edited

- `skills/zed/SKILL.md` — added Skill Version section, G7' (property/option additions discipline), G9 (PRC unifying principle), Session Memory subsection, Stable Names entries for Supporting Documents / Reminders / Surfaced Items / Time Sensitivity / version fields, Glossary updates, new Outline rows for Reminders + Recall + version-migration, Reference Manifest updates, Bootstrap step 0 (version constant safeguard) and step 7 (cross-plugin detection), v2 changelog entry, Current bumped to v2.
- `skills/zed/flows/briefing.md` — Step 1 source-scan extended for Reminders / Awaiting Deploy / Supporting Documents / legacy DBs. Step 2 Filter Rules gate extended to Surfaced Items state check. New "Reminders Today" section between Alerts and Carryover. New "Awaiting Deploy" subsection under Tasks. Composite ranking for Top 3 / Tasks. Once-per-session legacy nudge with deconflict.
- `skills/zed/flows/interactive.md` — sub-flow routing rewritten (Research / Drafts / Action Plans → Supporting Documents). New Reminders / version-migration routing rows. Suppress / handled / un-suppress / resurface verbs.
- `skills/zed/flows/setup.md` — S4 structure creation adds Supporting Documents and Reminders, removes the four legacy folders/databases for new users. S5 sets `last_seen_version = v2` and `migration_v2_choice = complete` for new users (no migration needed).
- `skills/zed/flows/file-cabinet.md` — added Sideways Entities section (Supporting Documents and Reminders sit sideways from the four-level chain).
- `skills/zed/flows/task-management.md` — Time Sensitivity inference at create with anchors, fuzzy-match against Reminders, Awaiting Deploy verbs, "shipped X" trigger, dangling-link cleanup on cancel.
- `skills/zed/flows/documentation-routing.md` — thin pointer to `references/documentation-routing.md` plus in-Zed routing wrapper. Adds Supporting Document Type routing as step 2.
- `skills/zed/flows/captains-log-recall.md` — RENAMED to `recall.md`. Old file overwritten with deprecation pointer (safe to delete after manual cleanup of any external references).
- `skills/zed/references/notion-conventions.md` — top-level structure adds Supporting Documents + Reminders databases, marks Research / Drafts / Action Plans / Reference as legacy. State Dashboard page-level properties add `Last Seen Version`, `Migration v2 Choice`, `Legacy Plugin Choice`. Body adds Surfaced Items + Filter Rules provenance. Tasks schema adds Time Sensitivity property + not-deployed Status option + saved view audit. New Supporting Documents and Reminders database schemas.
- `skills/zed/references/file-based-conventions.md` — folder layout adds Supporting-Documents/ and Reminders/, marks legacy folders. State Dashboard frontmatter adds version fields. Surfaced Items + Filter Rules sections in body. Task frontmatter adds `time_sensitivity` and `cos_id`, status enum adds not-deployed. New Supporting Document and Reminder frontmatter blocks. Logseq mappings updated. File-slug stability rule (`cos_id` UUID) added.
- `skills/zed/references/signal-filters.md` — filter pipeline rewritten with order. Canonical Identifiers section. Filter Rules provenance. Always Surface adds TS=5 hard override + Reminders + Awaiting Deploy. Composite ranking formula with worked examples. Time Sensitivity anchors (0–5). Reminders sort. Identifier-based dedup. Per-source read budget table.
- `skills/zed/references/watcher-playbook.md` — Weekly Review (Friday 4pm) gains v2 hooks: Awaiting Deploy summary, stale prep, stale brainstorm, cross-system per-project summary. Monthly Cleanup gains v2 hooks: Surfaced Items TTL, quarterly v2 migration re-prompt, tag promotion offer.

#### Files created

- `skills/zed/references/documentation-routing.md` — Type decision tree, Captain's Log graduation rule, Type-transition rules, legacy DB read rule, canonical tag list with normalization, G9 PRC summary.
- `skills/zed/flows/version-migration.md` — v1→v2 migration offer with three branches, auto-complete on zero, cross-plugin detection prompt with three answers.
- `skills/zed/flows/reminders.md` — Reminder CRUD, fuzzy-match-on-create, snooze, dismiss, terminal-action recurrence advance, lookahead trigger, dangling-link cleanup, G9 PRC summary.
- `skills/zed/flows/recall.md` — cross-system recall (Captain's Log + Supporting Documents + Briefings), tag normalization, Idea graduation offer at recall, `[legacy]` UX.

#### Out of scope (intentional)

- Older `ai-chief-of-staff:log` plugin in a separate plugin/repo NOT modified. Bootstrap step 7 detects parallel installation and prompts the user to uninstall.
- No forced data migration — existing entries in legacy databases (Research / Drafts / Action Plans / Reference) stay readable. v2 migration offer in `flows/version-migration.md` lets the user decide.

#### G7' audits required for v2 ship

- **Null-handling check.** Tasks reads handle null `time_sensitivity` (treated as 0 in composite). Tasks reads handle missing `cos_id` (UUID written on first read).
- **Status filter audit.** Briefing source-scan and task plate query exclude `not-deployed` from Tasks section, surface in Awaiting Deploy.
- **Notion saved views audit.** "Active Tasks" and "Tasks Due" views update filter to exclude `not-deployed`. New "Awaiting Deploy" view created.
- **External integration audit.** Watcher prompts referencing Tasks Status filters do NOT need updating (existing watchers filter to to-do/in-progress/blocked which already excludes not-deployed).

---

## How to push to GitHub

The changes above are saved locally. To sync to GitHub:

```
cd /Users/avigdorlevi/Avigdor's\ Vault/Projects/Chief\ of\ Staff/skill/zed-plugin
git add .
git commit -m "Zed v2 — 5-feature deployment (Mikoshi-validated 94/100)"
git push
```
