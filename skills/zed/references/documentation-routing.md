# Documentation Routing — Source of Truth

> **Loaded by:** `flows/documentation-routing.md`. Also referenced by `flows/interactive.md`, `flows/recall.md`, and `flows/briefing.md` (proactive documentation suggestions).
>
> **Why this file exists.** v2 consolidated four legacy databases (Research / Action Plans / Reference / Drafts) into a single Supporting Documents database with a Type field. This file holds the routing logic — the Type decision tree, the graduation rule, Type transitions, the legacy DB read rule, and the canonical tag list. Loaded once; referenced everywhere routing happens.

---

## Supporting Documents Type — decision tree

5 Types. Run top to bottom. First match wins. If nothing matches cleanly, ask the user from the 5 Types.

```
Is the user explicitly drafting outgoing correspondence (email, message)?
  → Type = artifact, Tag = draft

Is this evergreen reference material (SOP, playbook, saved link, reusable asset)?
  → Type = artifact, Tag = reference

Is this preparation for a specific upcoming meeting / event?
  → Type = prep

Is this analysis or research output (sub-agent finding, competitive scan, data review)?
  → Type = analysis (Tag = research if sub-agent generated)

Is this a step-by-step plan, strategy, or framework?
  → Type = plan (Tag = action-plan / framework / strategy as appropriate)

Is this exploratory thinking, an idea-stage working doc, a brainstorm tied to a project?
  → Type = brainstorm

None match → ask the user: "I'd file this as one of [brainstorm / prep / analysis / plan / artifact]. Which fits?"
```

Tell the user the Type you picked so they can correct it: "Saving as a `brainstorm` Supporting Doc tagged Marketing. Sound right?"

---

## Personal Journal Free Text vs Supporting Documents brainstorm — graduation rule (updated v3)

A persistent ambiguity: a personal reflection and a working brainstorm both feel like "thoughts to come back to." The skill keeps them separate.

- **Personal Journal Free Text** = personal reflection, mood/feelings/processing, may or may not touch project work. The user's voice, kept in a personal space.
- **Supporting Document brainstorm** = working doc tied to a specific Project, multi-paragraph or structured content expected, you'll come back to develop it.

**At write time:**
- User says "save this brainstorm" but no Project relation can be offered or inferred → ask the user where it belongs (Personal Journal entry, or pick a Project for a brainstorm). Don't silently default.
- User is journaling and the Free Text becomes multi-paragraph AND mentions an active Project → propose graduating that section to a Supporting Document brainstorm. The original journal entry stays intact (with a one-line pointer to the new SD).

**At recall time** (during `flows/recall.md`):
- If a Personal Journal entry's Free Text has grown to multi-paragraph AND the user is asking about it AND the topic ties to an active Project → offer once per recall: "This journal entry's free text has grown into something project-shaped — graduate to a Supporting Document brainstorm under [Project]?"
- If the user declines, don't push.

### Legacy graduation (Captain's Log Ideas, v3)

For users who have legacy Captain's Log Idea entries, the same recall-time graduation offer applies — Idea → Supporting Document brainstorm. This is preserved so users mid-migration aren't penalized for not having moved to Personal Journal yet. See `flows/version-migration.md` § v2→v3 for the migration paths.

---

## Type-transition rules

Working docs evolve. These rules keep Type meaningful.

- **prep → artifact.** A `prep` doc whose associated meeting/event date has passed by 7+ days: during weekly review, offer once per stale doc — "This prep doc is now post-event. Convert to artifact (kept as reference) or archive?"
- **brainstorm → plan.** When the user says "this is now the plan" / "let's go with this" on a brainstorm: ask — "Convert this brainstorm to a plan, or leave brainstorm and create a new plan that links to it?" Default: ask. Never silent.
- **artifact → archived.** Use `Tag = archived` as the marker. Briefing source-scan and recall queries exclude `Tag = archived`.
- **Personal Journal Free Text → SD brainstorm.** See graduation rule above (write-time when journaling, or recall-time). Replaces the v2 "Captain's Log Idea → brainstorm" path.
- **Captain's Log Idea → SD brainstorm (LEGACY, v3).** Still supported at recall time for users with legacy Captain's Log entries pre-v3 migration.

---

## Legacy DB read rule

The four legacy databases (Research / Action Plans / Reference / Drafts) are no longer written to in v2. They are still **read** from briefing source-scan and recall queries — entries are tagged `[legacy]` in the surfaced output.

**`[legacy]` UX:**

- Brief source-scan: "Source: Research DB *(legacy — consider migrating)*"
- Recall: "[legacy] Found 3 entries in Drafts: ..."
- Once per session, if any `[legacy]` entries surfaced AND `migration_v2_choice` ∈ {`leave-in-place`, `walking`}: the Chief offers once (not per item) — "I surfaced [N] legacy entries today. Want me to migrate any of them now?"
- **Deconflict (G9):** if the quarterly Monthly Cleanup re-prompt (from `flows/version-migration.md`) is already firing this session, suppress this per-session legacy nudge. At most one legacy-themed prompt per session. Track in session memory.

When `migration_v2_choice = complete` (no legacy entries left), this nudge stops automatically.

---

## Canonical tag list

Tag-filtered queries (e.g., "show me my personas," "pull up my drafts") match against this canonical list. Free-form tags are allowed but the canonical list is what's read by flows.

```
persona, framework, worksheet, draft, research, reference, action-plan,
meeting-prep, competitor, archived, legacy-uncategorized
```

### Normalization on write

When the agent writes a Tag to a Supporting Document:

1. Lowercase.
2. Replace whitespace and underscores with hyphens.
3. Match against the canonical list. If close (Levenshtein distance ≤ 2 from a canonical), normalize to canonical and tell the user once per session per non-canonical tag: "Storing as `persona` (you said 'personas')."
4. If no close match: store as user typed (lowercase, hyphenated). User can grow the canonical list over time by being consistent.

**Nag cap.** The "I normalized this for you" notification fires AT MOST ONCE per non-canonical tag per session (tracked in session memory). After that, the agent normalizes silently for the rest of the session.

### Normalization on read

Tag-filtered queries normalize the user's query string the same way before matching. Plus partial match: "show me my personas" → query Tag IN (`persona`, `personas`).

### Canonical list growth

When a non-canonical tag has been used 3+ times across Supporting Documents, the agent offers (during Monthly Cleanup): "You've used the tag `[X]` [N] times — promote to a canonical tag?" If yes, add to the canonical list (which lives here in this file — surface as a one-line edit suggestion to the user; cos-dev applies the actual edit).

---

## Where Supporting Documents live by platform

- **Notion:** `Supporting Documents` database under the Chief of Staff parent page. Schema in `notion-conventions.md`.
- **File-based:** `Chief-of-Staff/Supporting-Documents/` folder. Frontmatter in `file-based-conventions.md`.

---

## Snapshot rule

Supporting Documents are NOT part of the State Dashboard. G2 snapshot does NOT apply to creating/editing Supporting Document records. Applies only if the user's request also touches State Dashboard content.

Personal Journal entries do NOT require snapshots (they're not part of State Dashboard). Captain's Log entries (legacy) also do NOT require snapshots.

State Dashboard writes (Surfaced Items, Filter Rules, `last_seen_version`, `migration_v2_choice`, `migration_v3_choice`, `legacy_plugin_choice`, `personal_journal_location`, `journal_check_in_skips`, `journal_check_in_frequency`) DO require G2 snapshot.

---

## Forward-looking gate (G9 reference)

Every state-changing verb in this file (suppress, normalize, graduate, transition, archive) satisfies G9 (Provenance + Reversal + Conflict-Detection):

- **Provenance** — Filter Rule entries record `Source` (enum) + `Created At`. Tag normalization records the original input the first time.
- **Reversal** — every state change has an inverse: un-suppress, demote canonical tag, ungraduate, un-archive (set Tag back).
- **Conflict-Detection** — graduation and transitions ask before applying. Tag normalization tells the user what was changed.

See SKILL.md G9 for the rule.
