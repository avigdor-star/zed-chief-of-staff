# Flow: Reminders

**What this flow does:** Create, snooze, dismiss, convert, and recall Reminders. A Reminder is a date-scoped surfacing nudge, distinct from a Task (work to do).

**When to load it:** User says "set a reminder," "remind me to / about," "remind me later," "snooze this reminder," "dismiss this reminder," "what reminders," "show me my reminders," or any reminder management verb. Also loaded by `flows/interactive.md` when a Surfaced Items "remind me later" verb fires.

**Prerequisites:** Bootstrap complete. Universal Gates G1–G9 apply.

---

## Schema (Reminder records live in their own database/folder)

See `references/notion-conventions.md` (Notion) or `references/file-based-conventions.md` (file-based) for the literal schema. Key fields:

- `Name`, `surface_on` (date), `Recurrence` (none / daily / weekly / monthly / quarterly / yearly), optional `Project` / `Department` / `Domain` (a Reminder can attach to any level or be orphaned), `Status` (active / snoozed / dismissed), `snoozed_until` (date), `Source Item` (canonical identifier when created from Surfaced Items "remind me later"), `Linked Task` (optional relation), `Priority` (high / medium / low; default `medium`).

---

## Reminder creation

**Triggers:**
- "Set a reminder for [date]: [name]"
- "Remind me to [thing] on [date]"
- "Remind me about [thing] [later / next week / Friday / specific date]"
- "Remind me later about [item]" (from Surfaced Items context — Source Item canonical identifier prefilled)

**Steps:**

1. **Infer from context:** name, surface_on, project (if mentioned), priority.
2. **Resolve fuzzy dates:** "next week" → 7 days from today; "early next week" → next Monday; "Friday" → upcoming Friday. Always tell the user what you resolved: "Setting `surface_on` to 2026-05-04 (next Monday)."
3. **Vague punts** ("later," "not now") → default to 7 days. Tell the user: "Setting `surface_on` to 2026-05-03 (7 days). Want a different date?"
4. **Fuzzy duplicate check against active Tasks (G9 conflict-detection).** Run the fuzzy match per the rules below before creating. If a match is found in the same Project, surface the three options and let the user pick.
5. **Recurrence?** If the user mentioned "every Monday" / "weekly" / "monthly" / etc., set `Recurrence` accordingly. Otherwise default to `none`.
6. **Tell the user what you picked** so they can correct: "Reminder created: '[name]', surfaces 2026-05-04, recurrence weekly, attached to [Project]. Sound right?"

---

## Fuzzy duplicate check (Reminder ↔ Task)

Before creating a Reminder, fuzzy-match against active Tasks in the same Project. Same logic in reverse when creating a Task (handled in `flows/task-management.md`).

**Match rules:**

1. Tokenize Name: lowercase, drop stop words `{the, a, an, of, for, to, with, on, at, in, and, or, my, your, this, that}`.
2. **Match A:** 2+ remaining tokens shared between candidates.
3. **Match B:** Levenshtein distance ≤ 3 on the original lowercased Name string.
4. Trigger dedup prompt if A OR B AND both items are in same Project.

**Nag cap.** Don't fuzzy-prompt twice in the same session for the same target candidate. Track fired prompts in session memory.

**If a match is found, ask:**

> "There's already a Task called '[match]' in [Project]. Should this Reminder:
>   (a) replace the Task,
>   (b) link to it as a date-scoped surfacing nudge, or
>   (c) be separate?"

**Apply:**

- **(a) replace** — set the Task's Status to `cancelled` (preserves history). Create the new Reminder. Continue.
- **(b) link** — create the Reminder. Set `Linked Task` to the matching Task's `cos_id`. Continue.
- **(c) separate** — create the Reminder with no link. Continue.

---

## Reminder ↔ Task convert (one-step only)

Per the v2 plan, only one-step conversions are supported. Anything more complex is deferred.

- **"Convert this reminder to a task"** → create Task with same Name, Notes, Project. Set Reminder Status to `dismissed`. Recurrence is NOT carried (Tasks don't recur in this skill). Tell the user.
- **"This task is actually a reminder"** → create Reminder with `surface_on` = Task's Due Date. Set Task Status to `cancelled`. Ask: "What recurrence — none / daily / weekly / monthly / quarterly / yearly?" Tell the user when done.

---

## Snooze

When the user says "snooze this" on a Reminder surfacing in the brief, ask:

> "Snooze until — tomorrow / next week / specific date / custom?"

Set `snoozed_until` to the resolved date. Set Status to `snoozed`. Snooze does NOT advance Recurrence; it only delays the current occurrence.

When `snoozed_until ≤ today` on the next brief, Status flips back to `active` automatically and the Reminder surfaces.

---

## Dismiss / Got it / Mark done / Skip this one

These verbs are all terminal actions on a Reminder occurrence.

- For one-time Reminders (Recurrence = `none`): Status → `dismissed`. Reminder no longer surfaces.
- For recurring Reminders: advance `surface_on` to the next occurrence per the Recurrence rule. Status stays `active` for the next occurrence.

**"Skip this one"** is a synonym for "got it" — advances per Recurrence rule. Does NOT change the Recurrence type.

---

## Recurrence auto-advance (terminal action only)

A recurring Reminder advances `surface_on` to the next occurrence ONLY when the user takes a terminal action on it (`dismiss` / `got it` / `mark done` / `skip this one`).

- If the user **snoozes**, only the current occurrence is delayed; recurrence is untouched.
- If the user takes **no action** on the Reminder during the session, `surface_on` does NOT advance. The Reminder rolls into the next brief unchanged.

This means a recurring reminder rolls forward day-by-day until the user actually engages with it. Slightly more clutter, but no silent advance, no "I told you about that one" mismatches.

---

## Sort order in "Reminders Today" (briefing section)

The briefing's Reminders Today section sorts by:

1. **Overdue first** (`surface_on < today`), oldest `surface_on` first.
2. **Then today** (`surface_on = today`), grouped by Priority (high → medium → low).
3. **Then upcoming** (only if surfaced via lookahead — see below).
4. Within each Priority bucket, hierarchy as final tiebreaker (project-attached > department-level > domain-level > orphaned).

### Lookahead trigger

A Reminder with `surface_on > today` is included in "Reminders Today" only if:

- The user explicitly asks ("show me upcoming reminders" / "what's coming up this week"), OR
- The Reminder's `surface_on` is within 24 hours from now AND its Priority = `high`.

Otherwise upcoming Reminders wait for their `surface_on` date.

---

## Recall verbs

- "What reminders do I have?" → list active Reminders, sorted as above.
- "Show me upcoming reminders" → include lookahead window (see above).
- "What's snoozed?" → list reminders with Status = `snoozed`.
- "Show me dismissed reminders" → list reminders with Status = `dismissed` (history view).

---

## Dangling-link cleanup (when a linked Task is deleted/cancelled)

When a Task is deleted or its Status flipped to `cancelled` (handled in `flows/task-management.md`):

1. Scan Reminders for `Linked Task` matching the deleted/cancelled task's `cos_id`.
2. For each match: surface to the user once — "I cancelled Task '[name]'. Reminder '[reminder name]' was linked to it. Clear the link, or dismiss the reminder?"
3. **Clear the link** → null out `Linked Task` field. Reminder stays active. (No State Dashboard write — no G2 snapshot needed.)
4. **Dismiss** → Status → `dismissed` (preserves history; matches existing convention of not hard-deleting).

Same in reverse: when a Reminder is dismissed, scan for Tasks with reverse-links via Notion's auto back-relation; surface ambiguity if found.

---

## Snapshot rule

Reminders database is NOT part of the State Dashboard. G2 snapshot does NOT apply to Reminder CRUD. Applies only if the user's request also touches State Dashboard content.

---

## What this flow does NOT do

- Does NOT push to Apple Reminders or Google Calendar (out of scope for v2).
- Does NOT trigger alerts (no 3-tier cascade for Reminders — they're brief-only).
- Does NOT auto-create Reminders from Tasks with due dates (separate concepts; user creates each explicitly).
- Does NOT support custom recurrence (deferred to v2.1).

---

## G9 PRC summary for this flow

- **Provenance** — Reminder records `Source Item` when created from Surfaced Items. Linked Task field records the relation. Created date logged automatically.
- **Reversal** — every state change has an inverse: snooze ↔ unsnooze (snoozed_until clears at expiry); dismiss → recreate or undo dismiss; convert → convert back. Linking has cleanup procedure above.
- **Conflict-Detection** — fuzzy duplicate check at create time. Dangling-link cleanup at Task deletion.
