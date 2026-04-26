# Check-in State — Pacer Pattern Reference

> **Note:** The storage section on the State Dashboard is called "Check-in State." The frontmatter key that backs it is still `rollout_reminder:` for backward compatibility with existing dashboards. Both refer to the same data.

> **Applies to:** all four platforms (Obsidian / Logseq / markdown-folder / Notion).
>
> **Loaded when:** the Chief evaluates a rollout nudge, handles an Express-Lane request, handles a snooze, handles a contextual opening, or rolls back a watcher.
>
> **Companion files:** `SKILL.md` (briefing format + glossary), `watcher-playbook.md` (Validation Checklist, per-platform task identification), `file-based-conventions.md` / `notion-conventions.md` (storage schema).

## What this is

The Check-in State (also called the Rollout Reminder / Pacer pattern) is a **Pacer pattern** — a soft, once-per-window check-in that keeps the phased rollout alive without nagging. It is NOT an Axiom. Axioms (backup, structure) fire every session at the TOP of the briefing as urgent callouts. The Pacer fires at most once per window, at the BOTTOM of the briefing, as a single soft sentence.

The Chief does NOT ask the user to predict a cadence at setup. It calibrates from the user's observed responses using a three-bucket signal (overwhelming / just_right / ready_for_more).

## Storage schema (authoritative)

State is stored on the State Dashboard. The storage shape depends on the platform — see `file-based-conventions.md` (Obsidian / Logseq / markdown-folder) or `notion-conventions.md` (Notion) for the literal schema block.

**Fields (logical — same on all platforms):**

| Field | Type | Purpose |
|-------|------|---------|
| `last_mentioned` | date | Updated every time a scheduled ROLLOUT NUDGE fires. |
| `last_contextual_mention` | date or null | Updated every time a contextual opening fires (positive or negative hook). Independent of `last_mentioned`. Drives the contextual throttle. |
| `next_eligible` | date | **Authoritative** for scheduled check-ins. No other date field competes. |
| `last_checkin_response` | enum or null | One of: `overwhelming`, `just_right`, `ready_for_more`. Never `snooze` or `express` — those go in `last_action`. |
| `last_action` | enum or null | One of: `snooze`, `express`, or blank. |
| `express_partial` | bool | `true` if an Express Lane install was aborted mid-sequence. |
| `express_installed` | list | Watchers that completed with passing validation during a live Express Lane session. See "express_installed semantics" below for the full rule. |

### express_installed semantics

`express_installed` only tracks pieces installed **during a live Express Lane session** (a `yes` or `walk me through each one` path in Fix 2a). It is NOT a general install history.

- **Normal-install piece → rolled back** → `express_installed` is not touched (the piece was never in the list). Only `Automation Status` updates to `Not started`.
- **Express-Lane-install piece → rolled back** → remove the entry from `express_installed`. `Automation Status` also updates to `Not started`.
- **Piece installed outside the session (e.g., manually by the user) → rolled back** → same as normal-install. `express_installed` is not touched.

The list is "what's currently running AND was installed during a live Express Lane." On rollback of an Express-Lane piece, the entry is removed because the piece is no longer running.

## Decision logic (pseudocode)

### A. Briefing-time check — should a ROLLOUT NUDGE fire?

```
at end of briefing generation, after RECOMMENDED ACTIONS:
  if axiom_1_flagging OR axiom_2_flagging:
    skip nudge this session
    do NOT advance next_eligible
    exit
  if nudge_already_fired_this_session:
    skip nudge this session
    exit
  if next_eligible > today:
    skip nudge this session
    exit
  else:
    render ROLLOUT NUDGE section with the sentence template for the current situation
    set nudge_already_fired_this_session = true
    (the sentence fires once; state writes happen when the user responds)
```

### B. Day 7 first check-in — which template fires?

```
if this is the first rollout nudge (no prior last_mentioned value):
  if user still in Phase 0 (no watchers installed yet):
    use "First check-in (Day 7) — still in Phase 0" template
  else:
    use "First check-in (Day 7) — user already advanced on their own" template

note: Day 7 fires unconditionally with respect to rollout progress,
but the Axiom priority rule in (A) still applies — if an axiom is flagging,
the Day 7 nudge is suppressed and next_eligible does not advance.
Once the axiom clears, Day 7 fires on the very next briefing.
```

### C. Cadence self-calibration (response → next timer)

**Decision table — three-bucket check-in response:**

| Response | `last_checkin_response` | `next_eligible` | What the Chief does this session |
|----------|-------------------------|-----------------|----------------------------------|
| "overwhelming" (or any recognized overwhelm phrase) | `overwhelming` | today + **21 days** | Offer: simplify or roll back a piece |
| "just right" | `just_right` | today + **7 days** | Confirm, nothing else to do |
| "ready for more" | `ready_for_more` | today + 7 days (re-eligibility floor after Express offer) | **Suggest Express Lane this session** — speak the Express Lane confirmation template and branch per Section E |

```
on user response to a scheduled check-in:
  if response is "overwhelming" (or any recognized overwhelm phrase):
    last_checkin_response = overwhelming
    next_eligible = today + 21 days
    offer: simplify or roll back a piece
  if response is "just right":
    last_checkin_response = just_right
    next_eligible = today + 7 days
    confirm, nothing else to do
  if response is "ready for more":
    last_checkin_response = ready_for_more
    next_eligible = today + 7 days
    suggest Express Lane this session — speak the Express Lane
      confirmation template (see Verbatim Phrasings, Section E)
      and route into the three branches (yes / no / walk me through each one)
  always update last_mentioned = today
```

**Why these numbers.** Short loops. If the user says "overwhelming," 21 days is long enough to let the dust settle without feeling abandoned. "Just right" at 7 days keeps a weekly rhythm that matches the Weekly Review cycle. "Ready for more" goes directly to Express Lane because the user has already signaled appetite — making them wait another cycle to hear the offer is friction.

### D. Contextual-opening throttle

```
when a contextual hook is detected during a session (positive or negative):
  if explicit Express Lane intent ("install everything now"):
    not throttled — run Express Lane confirmation
  if explicit snooze intent ("hold off on all this"):
    not throttled — run Snooze confirmation
  if last_contextual_mention within last 7 days:
    skip — do NOT speak a contextual template this session
  else:
    speak the matching Fix 2b template (positive or negative)
    update last_contextual_mention = today
```

### E. Express Lane — the three branches

```
on Express Lane intent detected:
  speak the "Express Lane confirmation" template (see Verbatim Phrasings)

  on "yes":
    for each remaining watcher in rollout timeline:
      install the scheduled task
      run the matching validation from watcher-playbook.md Validation Checklist
      if validation fails: stop, report, treat as mid-abort
      confirm in plain English: "Installed. Test passed — [specific result]"
    if user interrupts with "stop" / "pause" / "not this one":
      goto mid-abort (F)

  on "no":
    last_action = express
    next_eligible = today + 7 days
    say: "Got it — staying gradual. I'll check back next week."

  on "walk me through each one":
    for each remaining watcher:
      speak the "Per-piece template (v3)" (see Verbatim Phrasings)
      on "install": install, validate, confirm, move on
      on "skip": move on (do NOT install, do NOT advance Automation Status for this piece)
      on "stop": goto mid-abort (F)
    after last piece:
      speak closing line (see Verbatim Phrasings)
```

### F. Mid-Express-Lane abort (Fix 4e)

```
on abort during either "yes" or "walk me through each one":
  express_partial = true
  express_installed = [the pieces that completed with passing validation]
  next_eligible = today + 21 days
  on next session's rollout nudge:
    say: "Last time we were installing [list]. We got through [completed list]
           before you paused. Want to finish the rest, hold here, or roll something back?"
```

### G. Snooze

```
on snooze intent detected:
  speak the "Snooze confirmation" template (see Verbatim Phrasings)
  on user-specified window (e.g., "two weeks", "a month"):
    next_eligible = today + window
    last_action = snooze
```

### H. Rollback (Fix 4f)

```
on rollback request (offered in Fix 4c or Fix 4e):
  user picks which piece to remove
  speak the "Rollback confirmation" template verbatim (see Verbatim Phrasings)
  on "yes":
    1. identify the scheduled task for that watcher (see watcher-playbook.md
       "Identifying a scheduled task for rollback" table for exact names).
    2. branch on WHERE the task runs:
       - Desktop watcher → DELETE_DESKTOP path (Chief deletes directly)
       - Cloud routine  → GUIDE_CLOUD path (Chief walks the user through manual deletion)
    3. update Automation Status in State Dashboard — set that component's Status to "Not started"
    4. if the piece is in express_installed: remove it from the list
    5. confirm: "Done. [Watcher name] is off. Automation Status updated."
  on "no":
    no state change
    ask: "Want to roll back a different piece, or hold?"

the verbatim confirmation step is NEVER skipped. This is the only Pacer action
that crosses the watcher-playbook.md scheduled-task boundary.
```

**DELETE_DESKTOP path (desktop watcher rollback) — Chief deletes directly.**

The Chief can reach desktop scheduled tasks through the Cowork scheduled-tasks tool. After the verbatim "Rollback confirmation" lands a `yes`:

1. Speak verbatim:

   > "Removing the desktop [watcher name] now."

2. Use the scheduled-tasks tool to delete the desktop task by name (per the watcher-playbook.md identification table).
3. If the deletion API returns an error: stop, surface verbatim:

   > "I couldn't delete the desktop [watcher name] — the scheduled-tasks tool returned [error]. Can you try removing it manually from the Cowork scheduled-tasks panel? Once it's gone, tell me 'removed' and I'll update the Automation Status."

4. On success: snapshot State Dashboard (hard gate), update Automation Status, remove from `express_installed` if present, and speak the verbatim "done" confirmation: "Done. [Watcher name] is off. Automation Status updated."

**GUIDE_CLOUD path (cloud routine rollback) — Chief cannot reach the cloud, walks the user through manual deletion.**

Cloud routines live on Anthropic's servers (claude.ai/code/scheduled). The Chief has no API to delete them. After the verbatim "Rollback confirmation" lands a `yes`:

1. Speak verbatim:

   > "I can't reach cloud routines from here, so you'll need to delete this one yourself. It takes about 30 seconds:
   >
   > 1. Open claude.ai/code/scheduled in your browser.
   > 2. Find the routine called [exact routine name from Automation Status — e.g. 'Email Watcher (Cloud)'].
   > 3. Click it, then click Delete (or the trash icon).
   > 4. Come back here and say 'deleted' so I can update the Automation Status.
   >
   > I'll wait."

2. Wait for the user to say `deleted` / `done` / `removed` / equivalent.
3. On confirmation: snapshot State Dashboard (hard gate), update Automation Status to `Not started`, remove from `express_installed` if present, and speak the verbatim "done" confirmation: "Done. [Watcher name] is off. Automation Status updated."
4. If the user says they couldn't find the routine or it's still showing as scheduled: speak verbatim:

   > "Tell me exactly what you see on claude.ai/code/scheduled — the names of every routine listed. I'll figure out which one to remove from there."

   Do NOT update Automation Status until the user confirms the routine is gone — otherwise the dashboard will lie about reality.

### I. 90-day max-quiet cap

```
on briefing-time check, if:
  next_eligible ≤ today AND today - last_mentioned ≥ 90 days:
    use the "Max-quiet cap (90 days)" sentence template
    (overrides the cadence-specific sentence for this one fire)
```

### J. Axiom priority override

```
on briefing-time check:
  if axiom_1_flagging (backup) OR axiom_2_flagging (structure):
    skip the rollout nudge this session, regardless of next_eligible
    do NOT advance next_eligible
    next session will re-evaluate
```

### K. Cadence regression (Fix 4c) — "too fast" mid-window

```
on any expression of overload BEFORE next_eligible (e.g., "this is too much"):
  treat as an immediate overwhelming response
  last_checkin_response = overwhelming
  next_eligible = today + 21 days
  speak verbatim:
    "Hearing that — let's slow down. I'll check back in three weeks, and in the meantime
     I can help roll back any piece that's weighing on you. Which one?"
```

### L. Manual advance without full Express (Fix 4b)

```
on phrases like "ready for more now", "speed up the timer", "let's add the next piece":
  treat as a ready_for_more response — NOT as Express Lane
  last_checkin_response = ready_for_more
  next_eligible = today
  offer the next single piece per rollout timeline
```

### M. Contextual throttle — explicit "stop nudging" phrases

Distinct from Section K (which fires on overload about the work itself). This fires when the user is specifically tired of the rollout reminders.

**Trigger phrases (examples — not exhaustive):**
- "stop nudging me"
- "stop nudging me about the rollout"
- "enough about the rollout"
- "I'll do it when I'm ready"
- "quit asking me about the watchers"
- "leave the rollout alone for a while"

```
on contextual throttle phrase detected:
  last_checkin_response = overwhelming
  next_eligible = today + 30 days
  last_mentioned = today
  confirm back in ONE sentence verbatim:
    "Got it — I'll back off the rollout reminders for a month. You can pick it up
     anytime by saying 'set up my next watcher' or 'install everything'."
  do NOT speak any further nudge or follow-up question this session.
```

**Why 30 days (not 21).** This is a louder, more explicit signal than the structured "overwhelming" bucket. The user is asking the topic to go away, not just saying the work feels heavy. 30 days gives a full month of silence — long enough that the user genuinely won't hear about it again unless they ask.

### N. Day 7 anchor robustness — `next_eligible` is a minimum, not a scheduled event

`next_eligible` is the **earliest date** a nudge may render. It is NOT a scheduled fire date.

This matters in three real-world cases:

1. **Day 7 lands on a weekend.** The user may not open a session that day. The nudge does not "fire and miss" — it simply waits. On the next session (Monday, or whenever) the render-gate in Section A re-evaluates: `next_eligible ≤ today` is now true, so the nudge fires then.
2. **The user takes a break and comes back days later.** Same behavior. The nudge holds for the next session and fires there. It does not pile up — only one nudge fires per session even if multiple days of eligibility have passed.
3. **An axiom is flagging the day `next_eligible` falls on.** Section J (axiom priority) applies: nudge is suppressed, `next_eligible` does not advance. As soon as the axiom clears, the nudge fires on the very next session.

**Rule (canonical):** the nudge fires on the next session where ALL render-gate conditions in Section A pass. Never on a clock date. Never twice in one session.

This means: do not wire any "fire on date X" trigger. The nudge is always evaluated at briefing-time inside the running session — there is no scheduler that fires it.

## Verbatim phrasings (pulled directly from spec v3 — do not paraphrase)

### Express Lane confirmation (Fix 2a opening)

> "I'm hearing you want to install all remaining pieces now — [list of watchers that would be added, e.g. 'email watcher, calendar watcher, end-of-day review, weekly review']. That skips the gradual baseline: if something breaks later, it'll be harder to figure out which piece caused it because everything got added at once. Want me to proceed anyway? (yes / no / walk me through each one)"

### Express Lane — "no" acknowledgement

> "Got it — staying gradual. I'll check back next week."

### Per-piece template (Fix 2a, v3 — NEW)

> "Next piece: [watcher name]. [One-line description from watcher-playbook.md, e.g. 'Scans your email every two hours and adds anything important to your Live Feed.'] Should I install this one now, skip it, or stop the walkthrough here?"

### Per-piece — install confirmation after validation

> "Installed. Test passed — [specific result, e.g. 'test entry appeared in your Live Feed']"

### Per-piece closing line (after last piece)

> "That's all of them. You're now at Phase [N] — up from Phase [previous]. Want a summary of what's running?"

### Contextual opening — negative hook (Fix 2b)

> "The [specific watcher] is designed for exactly this. Ready to set it up, or was that just venting?"

### Contextual opening — positive hook (Fix 2b)

> "Glad to hear that. If it feels right, we could add the [next watcher per rollout timeline]. Want to, or stay where you are?"

### Snooze confirmation (Fix 2c)

> "No problem. How long should I hold off on rollout nudges — a couple weeks, a month, or longer?"

### Cadence regression (Fix 4c)

> "Hearing that — let's slow down. I'll check back in three weeks, and in the meantime I can help roll back any piece that's weighing on you. Which one?"

### Contextual throttle — explicit stop-nudging (Section M)

> "Got it — I'll back off the rollout reminders for a month. You can pick it up anytime by saying 'set up my next watcher' or 'install everything'."

### Mid-Express-Lane abort — next-session recovery (Fix 4e)

> "Last time we were installing [list]. We got through [completed list] before you paused. Want to finish the rest, hold here, or roll something back?"

### Rollback confirmation (Fix 4f, v3 — NEW)

> "I'm going to remove the [watcher name] — that stops the automatic [what it did, e.g. 'email scans every two hours']. You can turn it back on any time. Go ahead?"

### Rollback — "done" confirmation

> "Done. [Watcher name] is off. Automation Status updated."

### Sentence templates for the ROLLOUT NUDGE section (Fix 3)

| Situation | Sentence |
|-----------|----------|
| First check-in (Day 7) — still in Phase 0 | "It's been a week since we set up your Chief of Staff. How's the rollout feeling — a bit overwhelming, just right, or ready for more?" |
| First check-in (Day 7) — user already advanced on their own | "You've moved faster than the default rollout pace — installed [list of watchers]. Quick pulse check on the rollout: feeling manageable, just right, or ready for even more?" |
| Regular check-in (`just_right` cadence, 7d) | "Quick weekly pulse on the watchers rollout — still feeling okay with the pace?" |
| Regular check-in (`ready_for_more` cadence, 7d floor) | "You said ready for more on the watchers rollout last time. Want to install all the remaining pieces at once (Express Lane), or add the next one and hold?" |
| Regular check-in (`overwhelming` cadence, 21d) | "Checking in lightly on the watchers rollout — still feeling overwhelmed, or easier now?" |
| Max-quiet cap (90 days) | "It's been a while since we talked about the rollout. Everything still working as-is, or want to add the next watcher?" |

## Intent detection — example phrases per trigger class

These are examples, not an exhaustive list. The Chief uses the **default-to-ask rule** (Fix 2d): if intent is unclear, ask rather than act. A false confirmation question costs 5 seconds. A false install costs trust.

| Trigger class | Example phrases |
|---------------|-----------------|
| Express Lane | "expedite onboarding", "set me up fully", "skip ahead", "do it all now", "install everything" |
| Contextual — negative hook | "drowning in email", "calendar chaos", "email is a mess", "I'm behind on everything" |
| Contextual — positive hook | "I finally feel on top of my email", "calendar feels manageable", "inbox is under control" |
| Snooze | "not yet", "hold off", "give me a break from this", "pause the rollout stuff" |
| Manual advance | "ready for more now", "speed up the timer", "let's add the next piece" |
| Cadence regression | "this is too much", "I'm overwhelmed again", "let's slow down" |
| Rollback | "roll back [watcher]", "turn off [watcher]", "remove the [watcher]" |
| Contextual throttle (Section M) | "stop nudging me", "enough about the rollout", "I'll do it when I'm ready", "quit asking about the watchers", "leave the rollout alone for a while" |

## Failure recovery — summary of all six paths

1. **4a Positive contextual openings** — "yes" advances `next_eligible` to today + installs; "no" silently moves `next_eligible` forward 2 weeks.
2. **4b Manual advance** — treated as `ready_for_more`, one piece at a time. NOT full Express Lane.
3. **4c Cadence regression** — mid-window overload → `overwhelming` + 21 days + offer rollback.
4. **4d Axiom priority** — Axiom 1 or 2 flagging → skip nudge, don't advance `next_eligible`.
5. **4e Mid-Express-Lane abort** — `express_partial = true`, record `express_installed`, `next_eligible` += 21 days, offer finish/hold/rollback next session.
6. **4f Rollback** — identify scheduled task, verbatim confirmation, delete task, update Automation Status to `Not started`, remove from `express_installed` if present, confirm.

## What this pattern does NOT do

- Does NOT replace the Weekly Review's ownership of the Automation Status table for installs. Weekly Review still updates Automation Status every Friday. The Pacer only writes to Automation Status on **rollback** — and only to set a component to `Not started`.
- Does NOT override the backup or structure axioms. Those always fire first.
- Does NOT create a new database or folder. All state lives on the existing State Dashboard.
- Does NOT read or write to the Live Feed.
- Does NOT send alerts (3-tier cascade). It is a single sentence in the briefing, never an email or DM.
