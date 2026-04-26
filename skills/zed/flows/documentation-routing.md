# Flow: Documentation Routing

**What this flow does:** Routes "log this," "document this," "record this," "note this down," "capture this," "journal this," "journal that," "remember this," "captain's log," and similar quick-documentation phrases to the right destination — task update, project note, State Dashboard field, or Captain's Log.

**When to load it:** User says any quick-documentation phrase. Loaded directly via the SKILL.md Outline at the start of a session, or invoked from `flows/interactive.md` mid-session.

**Prerequisites:** Bootstrap complete. Universal Gates G1-G8 from SKILL.md apply — especially G1 (approval) and G2 (snapshot for any State Dashboard touch).

---

## The strict routing order

Captain's Log is the last-resort fallback, NOT the default. The Captain's Log is a personal diary — the home for content that doesn't fit anywhere else. Route in this order:

### 1. Try operational first

Can this be a Task update, Project note, Department note, Watch List item, or State Dashboard field? If yes, propose placing it there.

Examples:
- "Sounds like this belongs as a note on the [Project X] record. Want me to add it there?"
- "That's a status change on [Task] — want me to mark it `in-progress` and add a note?"
- "Sounds like a Watch List item — want me to add 'keep an eye on [thing]'?"

### 2. If unsure — ALWAYS ask

If you can't confidently place the content in a specific task/project/dashboard location, ALWAYS ask the user where it should go. Never guess. Offer 2–3 specific options (including Captain's Log as one of them).

Example:
- "Not sure where this fits — is it a note on Project X, a new task under [Department], or a Captain's Log entry?"

### 3. Captain's Log is the explicit fallback, not the default

Only write to the Captain's Log when:
- (a) the user explicitly says "captain's log: …", "add to my captain's log …", or similar explicit journal phrasing; OR
- (b) the user picks Captain's Log from the options offered in step 2.

---

## Explicit Captain's Log phrases bypass routing

If the user says "captain's log: …" or "add to my captain's log …", write directly per `references/captains-log.md`. No routing needed — the user has been explicit.

- Infer **Type** (Milestone / Decision / Win / Learning / Problem / Idea) from context.
- Infer **Project** from context (drawn from `State Dashboard → Active Projects`, plus `Personal` and `Other`).
- Tell the user what you picked so they can correct it: "Logged as a Milestone tagged Chief of Staff. Sound right?"
- Confirm briefly after writing per the platform-specific confirmation pattern in `captains-log.md`.

---

## What to do if the user pushes back

If the user replies with where they actually want it ("no, that's a task" / "make it a project note" / "stick it in the dashboard"), help them route to that place. Don't argue — they know their content better than you do.

If the user changes their mind after you've written somewhere ("undo that, put it in the log instead"), undo the write (or leave a note about the move) and re-route.

---

## Snapshot rule

Any write to the State Dashboard requires a snapshot first per Universal Gate G2. Captain's Log entries do NOT require snapshots — they're not part of the State Dashboard. See `references/vault-safety.md` for the full snapshot mechanics and what's covered.
