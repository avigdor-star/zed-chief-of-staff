# Flow: Recall (Cross-System)

**What this flow does:** Queries Personal Journal, Supporting Documents, and Briefings for entries matching a user request. Also reads legacy Captain's Log entries if any exist (preserved across v3 migration). Returns matches interleaved by date with source annotation. Replaces the v1 `flows/captains-log-recall.md` (renamed in v2; queries updated for Personal Journal in v3).

**When to load it:** User asks recall questions like "what did I journal about X," "show me my wins," "what did I decide about Y," "pull up anything tagged [project]," "what have I written about X," "what did I think about X," "show me my personas," "pull up my drafts," "when did I feel overwhelmed," any Tag-filtered query, any mood-filtered query.

**Prerequisites:** Bootstrap complete. Personal Journal may or may not exist (auto-created on first journal use). Captain's Log may or may not exist (legacy — preserved across v3 migration). Supporting Documents may or may not exist depending on user's setup state. Briefings exist after the first briefing.

For schema details:
- Personal Journal → `references/personal-journal.md`
- Captain's Log (LEGACY, v3) → `references/captains-log.md`
- Supporting Documents → `references/documentation-routing.md` (Type tree, canonical tags) + `references/notion-conventions.md` / `references/file-based-conventions.md`
- Briefings → `references/notion-conventions.md` / `references/file-based-conventions.md`

---

## Single-system queries (specific recall)

If the user's question targets one system, query just that one. Don't expand the search.

| Pattern | Where |
|---------|-------|
| "Show me my wins" / "when did I feel [X]" / "my journal entries about [topic]" / "personal journal entries from [period]" | Personal Journal (also queries legacy Captain's Log if entries exist) |
| "Show me my personas" / "pull up my drafts" / "my brainstorms about X" | Supporting Documents only |
| "What was in last week's briefing" / "find that briefing about X" | Briefings only |
| "Captain's log entries about Y" (legacy phrasing) | Captain's Log only (legacy) — also offer: "These are legacy entries — your new entries live in Personal Journal." |

---

## Cross-system queries (broad recall)

If the user's question is open-ended ("what have I written about X" / "what did I think about X" / "everything about [topic]"), search ALL THREE systems and return interleaved by date.

### Search rules

- **Personal Journal:** full-text on all body sections (Wins / Challenges / Gratitude / Notes), plus optional Mood / Energy / Date filters. Mood is text — use partial match (e.g. "anxious" matches "anxious," "feeling anxious," "a bit anxious").
- **Captain's Log (LEGACY, v3):** full-text on Title + Details, plus optional Type / Project / Date filters. Surfaced with `[legacy CL]` annotation. Only queried if the legacy structure exists with entries.
- **Supporting Documents:** full-text on Name + body/Notes, plus Tag filter (normalized per `references/documentation-routing.md`), plus Related Projects / Related Departments / Related Tasks relations.
- **Briefings:** full-text on body. Briefings are dated; sort surfaces by Date.

### Tag normalization (Supporting Documents)

Before matching a Tag-filtered query against Supporting Documents:

1. Lowercase the user's tag string.
2. Replace whitespace and underscores with hyphens.
3. Match against canonical tags in `references/documentation-routing.md`.
4. Use partial match too: "show me my personas" → query Tag IN (`persona`, `personas`).

---

## Result format

For each matching entry, return:

- **Date**
- **Source annotation** — `[Journal]`, `[legacy CL]` (legacy Captain's Log), `[Supporting Doc]`, or `[Briefing]`. Add `[legacy]` prefix if the entry is in a v2-legacy DB (Research / Action Plans / Reference / Drafts).
- **Title / Name** — for Personal Journal entries with no explicit title, use the date.
- **Type / Mood** — for Personal Journal: Mood if present (e.g., `mood: calm`). For Captain's Log (legacy): Type (Milestone / Decision / Win / etc.). For Supporting Documents: Type (brainstorm / prep / analysis / plan / artifact).
- **One-line snippet** from body — first sentence or so

Format as a compact list, sorted by Date descending. Cross-system queries interleave all sources.

Example:

```
2026-04-27  [Journal]        mood: anxious — pre-launch jitters before the v3 ship
2026-04-26  [Supporting Doc] Brainstorm — Pricing Strategy
2026-04-22  [legacy CL]      Decision — Switched to per-seat pricing
2026-04-15  [Briefing]       Top 3 included pricing review
2026-04-10  [legacy] [Research] Competitor pricing scan
```

If many matches, show the 10 most recent and tell the user "there are [N] total — say 'show all' if you want the rest."

If nothing matches, say so plainly: "Nothing matches that. Want me to broaden the search?"

---

## Graduation offers during recall (v3)

Two graduation patterns can fire during recall — at most one per recall (G9 conflict-detection):

### Personal Journal Free Text → Supporting Document brainstorm

When the result set includes a Personal Journal entry whose Notes / Free Text section is multi-paragraph AND mentions an active Project, offer once per recall:

> "This journal entry's free text reads like project work for [Project] — graduate to a Supporting Document brainstorm under that project? (The journal entry stays intact with a pointer.)"

If the user says yes:
1. Create a new Supporting Document with `Type = brainstorm`, copy the Notes / Free Text content into the body.
2. Add a one-line pointer in the journal entry's Notes section: "Graduated to [link to SD] on [date]."
3. Confirm: "Graduated. Reflection stays in your journal; project thinking lives in the Supporting Doc."

### Legacy Captain's Log Idea → Supporting Document brainstorm

When the result set includes a legacy Captain's Log Idea entry that has grown to multi-paragraph AND the topic ties to an active Project, offer once per recall:

> "This Captain's Log Idea looks like it's grown — graduate to a Supporting Document brainstorm under [Project]? (Legacy entry stays intact with a pointer.)"

If yes:
1. Create a new Supporting Document with `Type = brainstorm`, copy the Idea's content into Notes / body.
2. Set the original Captain's Log entry's Details to a one-line note: "Graduated to [link to new SD] on [date]." Don't delete the original — preserve history.
3. Confirm: "Graduated. Original legacy entry kept with a pointer."

If declined on either, don't push.

---

## Legacy entries — `[legacy]` UX

Per `references/documentation-routing.md` § Legacy DB read rule:

- Legacy entries (Research / Action Plans / Reference / Drafts) are read by recall but tagged `[legacy]` in the surfaced output.
- Once per session, if any `[legacy]` entries surfaced AND `migration_v2_choice` ∈ {`leave-in-place`, `walking`} AND the v2-migration prompt is NOT firing this session: offer "I surfaced [N] legacy entries today. Want me to migrate any of them now?" (G9 conflict-detection: at most one legacy-themed prompt per session.)

---

## What this flow does NOT do

- Does NOT write to Personal Journal, Captain's Log, Supporting Documents, or Briefings (writes are handled by `flows/journal.md`, `flows/documentation-routing.md`, and the briefing flow).
- Does NOT snapshot the State Dashboard. Recall is a read; G2 doesn't apply.
- Does NOT hard-archive entries during recall. Archival is `Tag = archived` (Supporting Documents) or status-based (Tasks).

---

## Snapshot rule

This flow is read-only. G2 does not apply. The graduation offer above DOES write — but to Captain's Log and Supporting Documents, neither of which is part of State Dashboard. So no G2 snapshot needed for graduation either.

The legacy nudge offer, if accepted, can route the user to `flows/version-migration.md` which handles its own G2 snapshots when writing State Dashboard.
