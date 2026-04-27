# Signal Filters & Priority Ranking

## Filter pipeline (order matters)

Run candidate items through these checks in order. First match drops the item.

1. **User-Defined Filter Rules** (highest priority).
2. **Surfaced Items state check** (added v2) — drop if the item's canonical identifier (see § Canonical Identifiers below) matches an entry in `State Dashboard → Surfaced Items` with state ∈ {`suppress-forever`, `suppress-this-thread`, `handled`}. Items in state `active` continue through the pipeline.
3. **Always Surface / Skip rules** (below).

For `remind-later` semantics, see `flows/reminders.md` — a "remind me later" verb on a Surfaced Item creates a Reminder linked to the item via `Source Item` (canonical identifier). The original item still flows through normal filters.

## Canonical Identifiers (added v2)

Used by Surfaced Items state lookup, deduplication, Filter Rule provenance, and any "have I seen this before" check across briefs.

| Source | Identifier (canonical form) |
|--------|------------------------------|
| Email (Gmail) | `gmail:thread:<thread_id>` |
| Email (Outlook) | `outlook:thread:<conversation_id>` |
| Slack message | `slack:<channel_id>:<ts>` (parent thread ts) |
| Teams message | `teams:<chat_id>:<message_id>` |
| Calendar event | `cal:<provider>:<event_id>` (recurring instance: `cal:<provider>:<event_id>:<instance_date>`) |
| Task tracker item | `task:<provider>:<id>` |
| Internal Task (CoS DB) | `cos-task:<page_id>` (Notion) or `cos-task:<cos_id>` (file-based UUID) |
| Internal Reminder | `cos-reminder:<page_id>` or `cos-reminder:<cos_id>` |
| Supporting Document | `cos-doc:<page_id>` or `cos-doc:<cos_id>` |

**Mutation policy.** Thread/conversation IDs are sticky — subject edits, new participants, or reply-all chains do NOT generate a new ID. The conversation is the same; the ID is the same. If a downstream action splits a thread (rare; some clients do this), surface to the user — don't silently treat as new.

## User-Defined Filter Rules

The State Dashboard contains a **Filter Rules** section where the user lists senders, domains,
companies, and topics they NEVER want to see. These override everything else in this file.

**Before applying any other filter logic, check every candidate item against the Filter Rules
list.** If an item matches a filter rule — by sender name, email domain, company name, site
name, or topic — drop it immediately. Do not surface it in any briefing section, do not carry
it over from a previous briefing, do not include it in alerts. A filter rule added after an
item was first surfaced still kills that item on the next pass.

This check applies to ALL briefing sections: Top 3, Messages That Need You, Carryover,
Recommended Actions, Risks/Blockers, Watch List Check, **Reminders Today** (added v2), **Awaiting Deploy** (added v2).

**Filter Rules provenance (added v2).** Filter Rules table gains `Source` enum (`manual` / `suppress-forever` / `setup` / `migration`) and `Created At` columns so the user can audit and reverse rules created via "suppress forever" using the un-suppress verb (see `flows/interactive.md`).

## Always Surface

- Deadlines within 48 hours
- Unread emails from real people (not newsletters, not automated notifications)
- Calendar conflicts or back-to-back meetings with no prep time
- Tasks marked urgent or high priority in the task tracker
- **Tasks with `Time Sensitivity = 5`** (added v2 — hard override; surface in Top 3 regardless of Priority)
- Items flagged as risks or blockers in the previous briefing
- Items on the Watch List in the State Dashboard
- Anything from a person on the High-Priority People list in the State Dashboard
- Messages matching patterns in the Missed Messages section (senders or subjects previously missed)
- **Reminders where `Status = active AND surface_on ≤ today AND (snoozed_until is null OR snoozed_until ≤ today)`** (added v2 — surfaced in "Reminders Today" briefing section)
- **Tasks with `Status = not-deployed`** (added v2 — surfaced separately in "Awaiting Deploy" briefing subsection, NOT in normal Tasks section)

## Skip Unless Asked

- Marketing newsletters and promotional emails
- Automated tool notifications (GitHub bots, CI alerts, Airtable alerts, etc.)
- Tasks with no due date untouched for 2+ weeks (surface in weekly review only)
- Routine recurring calendar events (unless there's a conflict)

## Priority Ranking

When ranking importance, use this category order first:

1. **Hard deadlines** — things expiring within 48 hours
2. **People waiting** — someone is blocked until the user responds
3. **Revenue impact** — directly tied to sales, launches, or client delivery
4. **Strategic work** — moves the business forward but not time-sensitive
5. **Everything else**

If unsure whether something is urgent, ask the user. Do not guess.

### Within-category ranking — composite score (Tasks, added v2)

Within each category, rank Tasks by composite score:

```
priority_weight = {urgent: 4, high: 3, medium: 2, low: 1, null: 0}
ts_weight       = Time Sensitivity (0–5, null treated as 0)
composite       = (priority_weight × 2) + ts_weight

# Hard override: Time Sensitivity = 5 always lands in Top 3 regardless of Priority.
```

Worked examples:
- `urgent + TS=0` → composite 8.
- `low + TS=5` → composite 7 + hard override → still surfaces.
- `medium + TS=4` → composite 8, ties with `urgent + TS=0` — Priority breaks tie (urgent wins).
- `high + TS=3` → composite 9, beats `urgent + TS=0`.

### Time Sensitivity anchors (0–5)

Show the user inline at task-create time when asking about Time Sensitivity:

| TS | Meaning | Example |
|----|---------|---------|
| 0 | No time pressure | "Explore competitor landscape" |
| 1 | Soft this-month | "Update bio page" |
| 2 | Soft this-week | "Draft Q2 review" |
| 3 | Soft this-day | "Reply to vendor" |
| 4 | Hard same-day | "5pm school pickup" |
| 5 | Drop-everything-now (hard override) | "Site is down" / "Payment failed" |

Null/missing Time Sensitivity is acceptable — treated as 0 in the composite.

### Reminders sort (in "Reminders Today" briefing section)

1. **Overdue first** (`surface_on < today`), oldest `surface_on` first.
2. **Then today** (`surface_on = today`), grouped by Priority (high → medium → low).
3. **Then upcoming if surfaced via lookahead** — only included if user explicitly asked OR `surface_on` is within 24 hours AND Priority = `high`.
4. Hierarchy as final tiebreaker (project-attached > department-level > domain-level > orphaned).

## Email Filtering Details

**REPORT if:**
- Email is from a real person (not a newsletter, not automated)
- Email asks a question or requests action
- Email is from someone on the High-Priority People list
- Email mentions a deadline, payment, or urgent matter
- Email matches a pattern from the Missed Messages section

**SKIP if:**
- Newsletters, marketing emails, promotional content
- Automated notifications from tools (GitHub, Airtable, Supabase, CI/CD, etc.)
- Emails that are FYI-only with no action needed

## Deduplication (added v2 — identifier-based)

Before writing any item to the Live Feed, posting to a messaging channel, or surfacing in a brief, dedup against existing entries using the **canonical identifier** (see § Canonical Identifiers above).

Match logic:
1. Compute the candidate's canonical identifier.
2. Check Live Feed `## Feed` section for entries with the same identifier still marked NEW or CARRIED OVER.
3. Check the current brief's already-included items.
4. If match found, skip.

**Fallback:** if a source doesn't expose a stable identifier (rare), fall back to sender + subject (emails) or task Name (task tracker). Tag the entry with `dedup_method: fallback` for diagnostics.

## Per-source read budget (added v2)

Briefing source-scan caps to prevent any single source from blowing out the brief:

| Source | Default cap | Behavior on cap |
|--------|-------------|-----------------|
| Email | 25 messages | If more, surface "+N more" with a verb to expand. |
| Calendar | today + tomorrow only | No cap; bounded by date. |
| Tasks | 50 active | Filter by composite rank, surface top 25 in brief, list rest under "more tasks." |
| Messaging channel | 50 messages since last session | If more, summarize the surplus. |
| Captain's Log | 7 days back, all entries | No cap; bounded by date. |
| Supporting Documents | tied-to-active-projects only | No cap on count; cap on listed details. |
| Surfaced Items | all non-active states | No cap; usually small. |
| Reminders | active + today | No cap; bounded. |
| Legacy DBs | each capped at 10 entries | Surface "+N more legacy" with the once-per-session migration nudge. |

User can override caps via "show me everything from [source]" verb.
