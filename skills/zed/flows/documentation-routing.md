# Flow: Documentation Routing

**What this flow does:** Routes "log this," "document this," "record this," "note this down," "capture this," "journal this," "journal that," "remember this," and similar quick-documentation phrases to the right destination — Task update, Project note, Supporting Document (with Type), or State Dashboard field. ALWAYS asks the user where the content goes — there is no silent default in v3.

**When to load it:** User says any quick-documentation phrase (general "log this" / "remember this" / "document this" / "note this down"). Loaded directly via the SKILL.md Outline at the start of a session, or invoked from `flows/interactive.md` mid-session.

> **NOT this flow:** explicit personal/journaling phrases ("personal check-in," "journal entry," "personal journal: …", "how I'm feeling today") route to `flows/journal.md` directly per the SKILL.md Outline. This flow handles the ambiguous documentation phrases that could go anywhere.

**Prerequisites:** Bootstrap complete. Universal Gates G1-G9 from SKILL.md apply — especially G1 (approval), G2 (snapshot for any State Dashboard touch), G9 (Provenance + Reversal + Conflict-Detection).

---

## Source of truth

The Type decision tree, graduation rule, Type-transition rules, legacy DB read rule, and canonical tag list all live in **`references/documentation-routing.md`**. Load that reference and apply.

This flow file holds only the in-Zed routing wrapper around it.

---

## The strict routing order — ALWAYS ASK (v3)

In v3, the routing always asks the user where ambiguous content goes. There is no silent fallback to any single destination. Captain's Log is retired and Personal Journal is reserved for explicit journal phrases (which route to `flows/journal.md`, not here).

### 1. Try operational first

Can this be a Task update, Project note, Department note, Watch List item, or State Dashboard field? If yes, propose placing it there.

Examples:
- "Sounds like this belongs as a note on the [Project X] record. Want me to add it there?"
- "That's a status change on [Task] — want me to mark it `in-progress` and add a note?"
- "Sounds like a Watch List item — want me to add 'keep an eye on [thing]'?"

### 2. Try Supporting Document (added v2)

If the content is a working doc — brainstorm, prep notes, plan, persona, framework, draft, analysis — apply the Type decision tree from `references/documentation-routing.md` and propose creating a Supporting Document. Examples:

- "Save this brainstorm" → Type = `brainstorm`. Ask for Project relation if not obvious.
- "I worked out a framework for X" → Type = `plan`, Tag = `framework`.
- "Prep notes for tomorrow's [meeting]" → Type = `prep`. Ask for Related Project if not implied.
- "Draft this as an email to [person]" → Type = `artifact`, Tag = `draft`.
- "Save this persona" → Type = `artifact`, Tag = `persona`.

Tell the user what Type you picked: "Saving as a `brainstorm` Supporting Doc tagged Marketing. Sound right?"

**Tag normalization (G9 provenance).** Tags written here go through the canonical normalization rules in `references/documentation-routing.md` § Canonical tag list. The "I normalized this" notification fires at most once per non-canonical tag per session.

### 3. ALWAYS ask if you can't place it confidently

If you can't confidently place the content in a specific Task / Project / Supporting Doc / Dashboard location, ALWAYS ask the user where it should go. Never guess. Never silently default to anywhere.

Offer 2–4 specific options. Personal Journal is one possible option ONLY if the content is reflective/personal in nature. Don't offer Personal Journal for operational content.

Example offers:
- "Not sure where this fits — is it a `brainstorm` Supporting Doc under [Project], a new task under [Department], or a Personal Journal entry?"
- "This could be: (a) a note on [Project], (b) a `prep` Supporting Doc, or (c) a State Dashboard Watch List item. Which?"
- "Looks reflective — Personal Journal entry, or `brainstorm` Supporting Doc tied to [Project]?"

If the user picks Personal Journal, hand off to `flows/journal.md` to capture per its schema (Date / Mood / Energy / Wins / Challenges / Gratitude / Free text).

---

## Legacy phrase redirect — "captain's log:" (v3)

Captain's Log is retired in v3 — but users may have years of muscle memory saying "captain's log: …" out of habit. Don't ignore the phrase; redirect gracefully:

If the user says "captain's log: …" or "add to my captain's log …" or any explicit Captain's Log phrasing:

> "Captain's Log is retired in v3 — Personal Journal is the new home for personal/reflective content. Want me to put this in your Personal Journal instead? (yes / no, route somewhere else)"

- **yes** → hand off to `flows/journal.md` with the supplied content as the Free Text seed. The user can add Mood / Energy / etc. or just save the free text.
- **no, route somewhere else** → return to Step 1 of this flow's strict routing order (operational → Supporting Doc → ask).

**Redirect frequency.** The redirect message fires every time the user says the legacy phrase — it's not a one-time prompt. The friction is intentional: it teaches the new vocabulary. After the user has accepted the redirect 3+ times in a single session, optionally append: "Tip: just say 'journal entry' or 'personal journal: …' next time to skip this question." (Tracked in session memory.)

---

## What to do if the user pushes back

If the user replies with where they actually want it ("no, that's a task" / "make it a Supporting Doc" / "stick it in the dashboard" / "actually put it in my journal"), help them route to that place. Don't argue — they know their content better than you do.

If the user changes their mind after you've written somewhere ("undo that, put it somewhere else"), undo the write (G9 reversal — every state-changing verb has an inverse) or leave a note about the move and re-route.

---

## Snapshot rule

Any write to the State Dashboard requires a snapshot first per Universal Gate G2. Personal Journal entries, Captain's Log entries (legacy), and Supporting Documents do NOT require snapshots — they're not part of the State Dashboard. See `references/vault-safety.md` for the full snapshot mechanics.

The Personal Journal first-use creation step (in `flows/journal.md` Step 1) DOES require G2 — it writes the journal location to Stable Names in the State Dashboard.

---

## G9 PRC summary for this flow

- **Provenance.** Every Supporting Document records `Date` (creation timestamp), `Tags` (normalized), and relations. Personal Journal entries record `Date` and `cos_id` (UUID). Operational updates record the source of the change in Task / Project notes.
- **Reversal.** Every routing decision has an inverse: re-route ("undo that, put it somewhere else"), graduate (Personal Journal Free Text → SD brainstorm), un-graduate (back).
- **Conflict-Detection.** When in doubt, ask. Never write to Personal Journal for operational content. Never write to a Supporting Document for purely reflective content. Tag normalization tells the user what was changed. Legacy "captain's log:" phrase is gracefully redirected (not silently rerouted).
