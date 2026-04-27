# Flow: Recall (Cross-System)

**What this flow does:** Queries Captain's Log, Supporting Documents, and Briefings for entries matching a user request. Returns matches interleaved by date with source annotation. Replaces the v1 `flows/captains-log-recall.md` (renamed in v2 — see SKILL.md G7 migration note).

**When to load it:** User asks recall questions like "what did I log about X," "show me my wins," "what did I decide about Y," "pull up anything tagged [project]," "what have I written about X," "what did I think about X," "show me my personas," "pull up my drafts," any Tag-filtered query.

**Prerequisites:** Bootstrap complete. Captain's Log exists (created during setup S4.5). Supporting Documents may or may not exist depending on user's setup state. Briefings exist after the first briefing.

For schema details:
- Captain's Log → `references/captains-log.md`
- Supporting Documents → `references/documentation-routing.md` (Type tree, canonical tags) + `references/notion-conventions.md` / `references/file-based-conventions.md`
- Briefings → `references/notion-conventions.md` / `references/file-based-conventions.md`

---

## Single-system queries (specific recall)

If the user's question targets one system, query just that one. Don't expand the search.

| Pattern | Where |
|---------|-------|
| "Show me my wins" / "what did I decide about X" / "captain's log entries about Y" | Captain's Log only |
| "Show me my personas" / "pull up my drafts" / "my brainstorms about X" | Supporting Documents only |
| "What was in last week's briefing" / "find that briefing about X" | Briefings only |

---

## Cross-system queries (broad recall)

If the user's question is open-ended ("what have I written about X" / "what did I think about X" / "everything about [topic]"), search ALL THREE systems and return interleaved by date.

### Search rules

- **Captain's Log:** full-text on Title + Details, plus optional Type / Project / Date filters.
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
- **Source annotation** — `[Captain's Log]`, `[Supporting Doc]`, or `[Briefing]`. Add `[legacy]` prefix if the entry is in a legacy DB (Research / Action Plans / Reference / Drafts).
- **Title / Name**
- **Type** (for Captain's Log: Milestone / Decision / Win / etc. For Supporting Documents: brainstorm / prep / analysis / plan / artifact)
- **One-line snippet** from Details / body — first sentence or so

Format as a compact list, sorted by Date descending. Cross-system queries interleave all sources.

Example:

```
2026-04-26  [Supporting Doc] Brainstorm — Pricing Strategy
2026-04-22  [Captain's Log]  Decision — Switched to per-seat pricing
2026-04-15  [Briefing]       Top 3 included pricing review
2026-04-10  [legacy] [Research] Competitor pricing scan
```

If many matches, show the 10 most recent and tell the user "there are [N] total — say 'show all' if you want the rest."

If nothing matches, say so plainly: "Nothing matches that. Want me to broaden the search?"

---

## Captain's Log Idea graduation offer (during recall)

When the result set includes a Captain's Log Idea entry that has grown to multi-paragraph AND the topic ties to an active Project, offer once per recall (not per item):

> "This Idea looks like it's grown — graduate to a Supporting Document brainstorm under [Project]?"

If the user says yes:
1. Create a new Supporting Document with `Type = brainstorm`, copy the Idea's content into Notes / body.
2. Set the original Captain's Log entry's Details to a one-line note: "Graduated to [link to new SD] on [date]." Don't delete the original — preserve history.
3. Confirm: "Graduated. Original entry kept with a pointer."

If declined, don't push.

---

## Legacy entries — `[legacy]` UX

Per `references/documentation-routing.md` § Legacy DB read rule:

- Legacy entries (Research / Action Plans / Reference / Drafts) are read by recall but tagged `[legacy]` in the surfaced output.
- Once per session, if any `[legacy]` entries surfaced AND `migration_v2_choice` ∈ {`leave-in-place`, `walking`} AND the v2-migration prompt is NOT firing this session: offer "I surfaced [N] legacy entries today. Want me to migrate any of them now?" (G9 conflict-detection: at most one legacy-themed prompt per session.)

---

## What this flow does NOT do

- Does NOT write to Captain's Log, Supporting Documents, or Briefings (writes are handled by `flows/documentation-routing.md` and the briefing flow).
- Does NOT snapshot the State Dashboard. Recall is a read; G2 doesn't apply.
- Does NOT hard-archive entries during recall. Archival is `Tag = archived` (Supporting Documents) or status-based (Tasks).

---

## Snapshot rule

This flow is read-only. G2 does not apply. The graduation offer above DOES write — but to Captain's Log and Supporting Documents, neither of which is part of State Dashboard. So no G2 snapshot needed for graduation either.

The legacy nudge offer, if accepted, can route the user to `flows/version-migration.md` which handles its own G2 snapshots when writing State Dashboard.
