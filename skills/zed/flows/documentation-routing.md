# Flow: Documentation Routing

**What this flow does:** Routes "log this," "document this," "record this," "note this down," "capture this," "journal this," "journal that," "remember this," "captain's log," and similar quick-documentation phrases to the right destination — Task update, Project note, Supporting Document (with Type), State Dashboard field, or Captain's Log.

**When to load it:** User says any quick-documentation phrase. Loaded directly via the SKILL.md Outline at the start of a session, or invoked from `flows/interactive.md` mid-session.

**Prerequisites:** Bootstrap complete. Universal Gates G1-G9 from SKILL.md apply — especially G1 (approval), G2 (snapshot for any State Dashboard touch), G9 (Provenance + Reversal + Conflict-Detection).

---

## Source of truth

The Type decision tree, Captain's Log graduation rule, Type-transition rules, legacy DB read rule, and canonical tag list all live in **`references/documentation-routing.md`**. Load that reference and apply.

This flow file holds only the in-Zed routing wrapper around it.

---

## The strict routing order

Captain's Log is the last-resort fallback, NOT the default. The Captain's Log is a personal diary — the home for reflective content that doesn't fit anywhere else. Route in this order:

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

### 3. If unsure — ALWAYS ask

If you can't confidently place the content in a specific task / project / Supporting Doc / dashboard location, ALWAYS ask the user where it should go. Never guess. Offer 2–3 specific options (including Captain's Log as one of them, but only if the content fits Captain's Log per the graduation rule).

Example:
- "Not sure where this fits — is it a `brainstorm` Supporting Doc under [Project], a new task under [Department], or a Captain's Log entry?"

### 4. Captain's Log is the explicit fallback, not the default

Only write to the Captain's Log when:
- (a) the user explicitly says "captain's log: …", "add to my captain's log …", or similar explicit journal phrasing; OR
- (b) the user picks Captain's Log from the options offered in step 3; OR
- (c) the content fits the graduation rule for Captain's Log Idea (no Project context, fleeting thought) per `references/documentation-routing.md` § Captain's Log Idea vs Supporting Documents brainstorm.

---

## Explicit Captain's Log phrases bypass routing

If the user says "captain's log: …" or "add to my captain's log …", write directly per `references/captains-log.md`. No routing needed — the user has been explicit. (Graduation offer still applies if applicable — see below.)

- Infer **Type** (Milestone / Decision / Win / Learning / Problem / Idea) from context.
- Infer **Project** from context (drawn from `State Dashboard → Active Projects`, plus `Personal` and `Other`).
- Tell the user what you picked so they can correct it: "Logged as a Milestone tagged [Project]. Sound right?"
- Confirm briefly after writing per the platform-specific confirmation pattern in `captains-log.md`.

**Graduation check (added v2).** If the user says "log this idea" but the content is multi-paragraph AND tied to an active Project, propose graduating to a Supporting Document brainstorm before writing to Captain's Log. See `references/documentation-routing.md` § Captain's Log Idea vs Supporting Documents brainstorm.

---

## What to do if the user pushes back

If the user replies with where they actually want it ("no, that's a task" / "make it a Supporting Doc" / "stick it in the dashboard" / "actually put it in the log"), help them route to that place. Don't argue — they know their content better than you do.

If the user changes their mind after you've written somewhere ("undo that, put it in the log instead"), undo the write (G9 reversal — every state-changing verb has an inverse) or leave a note about the move and re-route.

---

## Snapshot rule

Any write to the State Dashboard requires a snapshot first per Universal Gate G2. Captain's Log entries and Supporting Documents do NOT require snapshots — they're not part of the State Dashboard. See `references/vault-safety.md` for the full snapshot mechanics.

---

## G9 PRC summary for this flow

- **Provenance.** Every Supporting Document records `Date` (creation timestamp), `Tags` (normalized), and relations. Captain's Log records Type and Project.
- **Reversal.** Every routing decision has an inverse: re-route ("undo that, put it in the log instead"), graduate (Idea → brainstorm), un-graduate (back).
- **Conflict-Detection.** When in doubt, ask. Never write to Captain's Log when the content fits a working doc (graduation rule). Tag normalization tells the user what was changed.
