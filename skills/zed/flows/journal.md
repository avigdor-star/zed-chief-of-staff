# Flow: Personal Journal

**What this flow does:** Handles personal journaling — engaging the
end-of-brief Personal Check-in, writing entries directly via journal
phrases, and offering the four ways the user can journal (draft now / find
calendar time / blank entry / suggest a prompt).

**When to load it:** User says "personal check-in," "journal entry,"
"journal entry for today," "personal journal: …", "I want to journal,"
"how I'm feeling today," "my mood today," or any equivalent journaling
phrase. ALSO loaded by `flows/briefing.md` when the user engages the
end-of-brief Personal Check-in section.

**Prerequisites:** Bootstrap complete. Universal Gates G1–G9 from SKILL.md
apply throughout — especially G1 (approval) and G9 (PRC).

For schema, storage layout per platform, first-use creation behavior, and
recall — see `references/personal-journal.md`.

---

## Step 1 — Confirm Personal Journal exists (auto-create on first use)

Before anything else, check whether the Personal Journal structure exists at
the expected location for the user's platform (per
`references/personal-journal.md` § "Where Personal Journal lives — by
platform").

- **If it exists** → proceed to Step 2.
- **If it doesn't exist** → this is first use. Tell the user once:
  > "Looks like this is your first journal entry — I'll set up your Personal
  > Journal in [platform] and then we'll capture today's. OK?"
  
  Wait for explicit approval (G1). On yes:
  1. Snapshot State Dashboard first per G2 (we'll record the journal
     location in Stable Names).
  2. Create the structure per `references/personal-journal.md` § first-use
     creation behavior.
  3. Update `State Dashboard → My Setup → personal_journal_location` with
     the path / Notion page link.
  4. Confirm: "Done. Your Personal Journal lives at [location]."
  5. Proceed to Step 2.
  
  On no → drop the topic gracefully. Don't push.

---

## Step 2 — Ask how the user wants to journal (multiple choice)

Personality-aware tone per `references/personality.md`. Default wording
(adapt to chosen personality):

> "How do you want to journal today? Pick one:
>
> 1. **Draft it now** — I'll ask a few prompts, you talk it out, I'll write it up and save it.
> 2. **Find calendar time** — I'll scan your calendar for an open block and suggest a journaling slot.
> 3. **Blank entry** — I'll create a blank entry for today that you can fill in later.
> 4. **Suggest a prompt** — based on what's been happening this week, I'll propose something to write about."

**If the user just supplies content directly** (e.g. "Personal journal:
felt overwhelmed today, energy crashed after the Stripe call"), skip the
multiple choice and go straight to Step 3.4 (write entry from supplied
content).

---

## Step 3 — Apply the user's pick

### Pick 1 — Draft it now (talk it out)

Walk through the entry fields one at a time. Don't ask all 7 in a row —
that feels like a form. Ask conversationally, one or two at a time, and let
the user skip any field by saying "skip" or "nothing for that."

Suggested order (adapt to flow of conversation):

1. **Mood.** "How are you feeling today? One word or a short phrase is fine."
2. **Energy.** "Energy level — low, medium, or high?"
3. **Wins.** "Anything go well today?"
4. **Challenges.** "Anything hard?"
5. **Gratitude.** "Anything you're grateful for?"
6. **Free text.** "Anything else you want to capture? Open page."

After collecting whatever the user shares, write the entry per
`references/personal-journal.md` § entry schema. Use the platform-specific
file/page format per § Where Personal Journal lives. Confirm per §
Confirmation pattern.

**Snapshot rule.** Personal Journal entries are NOT part of State Dashboard
— G2 does not apply to entry writes. Only the first-use creation in Step 1
needs G2 (because it touches Stable Names location).

### Pick 2 — Find calendar time

1. Scan the user's calendar for the next 48 hours.
2. Look for open blocks of 20+ minutes that aren't during early morning
   (before 8am) or late evening (after 9pm) — pick something realistic.
3. Propose specific options:
   > "You're free at:
   > - Today 4:30–5:00 PM
   > - Tomorrow 8:00–8:30 AM
   > - Tomorrow 2:00–2:30 PM
   > 
   > Want me to block one of those for journaling?"
4. On the user's pick, create a calendar event titled `Personal journal time`
   for the chosen slot (per G6 trusted actions, calendar event creation in
   service of an explicit request is allowed).
5. Confirm: "Blocked [day] [time] for journaling. I'll be ready to draft
   when you sit down."

If the user picks none of the offered slots → offer to look further out, or
drop the topic.

### Pick 3 — Blank entry

1. Create a blank entry for today per the platform-specific format. All
   fields except Date are blank — the user fills in later.
2. Confirm with the file path / Notion link so the user can open it
   directly:
   > "Blank entry created at [location]. Open it whenever you want to
   > write."
3. Don't follow up. The user knows where it is.

### Pick 4 — Suggest a prompt based on the week

1. Look at the last 7 days of context the Chief has access to:
   - Captain's Log entries (if any — legacy)
   - Personal Journal entries from the last 7 days (if any)
   - Recent Tasks completed / closed (sense of accomplishment or grind)
   - Calendar density (busy week vs. light week)
   - Surfaced high-priority Top 3 items repeated across briefings
     (recurring themes)
2. Pick ONE prompt tailored to what's been going on. Examples:
   - Heavy week: "It's been a heavy week — three big calls plus the v2
     ship. What's still loud in your head?"
   - Quiet week: "Lighter week than usual. What do you want to do with that?"
   - Recurring theme: "[Topic X] keeps showing up — Top 3 four briefings in
     a row. What's underneath it?"
   - First entry ever: "Easy first one — what's your favorite thing about
     this week?"
3. After offering the prompt, ask: "Want to talk through that now, or save
   it for later?"
4. If "now" → fall through to Pick 1 (Draft it now), starting with the
   suggested prompt as the Free Text seed.
5. If "later" → create a blank entry with the prompt at the top of the Free
   Text section as a placeholder, so the user sees it when they open the
   entry.

---

## Step 4 — Engagement-skip tracking (soft mute)

Track engagement with the end-of-brief Personal Check-in across recent
briefings using `State Dashboard → My Setup → journal_check_in_skips`
counter:

- User engages (any pick above, including just supplying content) → reset
  counter to 0.
- User skips ("nope" / "skip" / "nothing today" / "next") → increment by 1.
- When counter reaches 5 → ask once at the next end-of-brief check-in:
  > "Heads up — you've skipped the Personal Check-in five times in a row.
  > Want me to dial it back to weekly, or pause it entirely? (weekly /
  > pause / leave-as-is)"
- Apply the user's pick to `State Dashboard → My Setup →
  journal_check_in_frequency` (`every-brief` / `weekly` / `paused`).
  Snapshot first per G2.
- After the answer (whatever it is), reset the skip counter to 0.

The briefing flow's end-of-brief Personal Check-in section reads
`journal_check_in_frequency` to decide whether to render this brief:
- `every-brief` → render every time (default)
- `weekly` → render only on the first brief of each calendar week (Monday)
- `paused` → don't render at all

The user can re-enable any time by saying "turn journal check-in back on"
or "personal check-in" — both reset frequency to `every-brief`.

---

## Step 5 — Graduation offer (G9 conflict-detection)

After writing any entry, check the Free Text content. If it's
multi-paragraph AND mentions an active Project by name, offer once:

> "That last bit reads like it's about [Project] — want me to also save it
> as a `brainstorm` Supporting Document under that project, so you can
> develop the thinking there? (yes / no)"

On yes:
1. Create a Supporting Document with `Type = brainstorm`, body = the Free
   Text content from the journal entry.
2. Add a one-line pointer to the journal entry: "Brainstorm started in
   journal — [link to entry]."
3. Add a one-line pointer in the journal entry's Free Text: "Graduated to
   [link to Supporting Doc] on [date]."
4. Confirm: "Saved both. Reflection stays in your journal; the project
   thinking lives in the Supporting Doc."

On no → don't push. The journal entry stands alone.

See `references/documentation-routing.md` § graduation rule for full
mechanics — this flow uses the same pattern Captain's Log Ideas used in v2.

---

## What this flow does NOT do

- Does NOT proactively suggest journaling mid-brief (only the dedicated
  end-of-brief Personal Check-in section). Captain's Log's "in-brief
  proactive prompts" pattern is intentionally retired in v3.
- Does NOT route operational content. "Log this task," "remember this for
  the project," etc. → handled by `flows/documentation-routing.md`, which
  asks the user where it goes.
- Does NOT auto-write entries. Entries are always the user's voice and
  always require explicit content from the user.
- Does NOT snapshot State Dashboard for entry writes (entries aren't part
  of State Dashboard). Only the first-use creation step touches Stable
  Names location and requires G2.

---

## G9 PRC summary for this flow

- **Provenance.** Entries record Date and `cos_id`. Created-by is always the
  user. Engagement skip counter is tracked per session and persisted to
  State Dashboard with snapshots.
- **Reversal.** Every pick has an inverse:
  - Pick 1 entry → editable / deletable
  - Pick 2 calendar block → user can delete the calendar event
  - Pick 3 blank entry → editable / deletable
  - Pick 4 prompt-then-blank → editable / deletable
  - Soft mute decision → reversible by saying "turn journal check-in back on"
  - Graduation → reversible (delete the Supporting Document; original
    journal entry preserved)
- **Conflict-Detection.** The graduation offer is the single conflict point
  — the Chief asks before moving content from the journal to a Supporting
  Document. Engagement skip tracking deconflicts the soft-mute prompt
  against other end-of-brief prompts (at most one frequency-discussion
  per brief).
