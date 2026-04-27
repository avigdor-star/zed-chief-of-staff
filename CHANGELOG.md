# Zed Plugin — Changelog

Changes to the skill during beta development. Newest first.

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
