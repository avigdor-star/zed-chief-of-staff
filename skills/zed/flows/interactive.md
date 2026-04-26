# Flow: Interactive (Mode B)

**What this flow does:** Dispatcher for any non-briefing request. Picks which sub-flow handles it, or handles inline if the request is short.

**When to load it:** Any user request that isn't a briefing, isn't a documentation phrase, and isn't first-time setup. The Outline routes most non-briefing things directly to specialized flows; this file catches everything else.

**Prerequisites:** Bootstrap complete. Universal Gates G1-G8 from SKILL.md apply throughout — especially G1 (approval), G2 (snapshot), G3 (Filter Rules), G6 (trusted actions).

---

## Quick task dispatch

- **Quick tasks** (draft, lookup, simple answer) → handle directly.
- **Medium tasks** (<5 min of work) → hand off to a background helper (sub-agent) with context from the State Dashboard.
  - Research → save to the Research folder/database (per the chosen platform's conventions)
  - Drafts → save to the Drafts folder/database
  - Action plans → save to the Action-Plans folder/database
  - Bug investigation → use the `bugfix-protocol` skill if available
  - Code review → use the `code-health-review` skill if available
- **Big tasks** (>5 min) → suggest a fresh Cowork session.

**All output documents must follow the conventions for the chosen platform** — see `references/file-based-conventions.md` (Obsidian / Logseq / markdown folder) or `references/notion-conventions.md` (Notion).

After a helper finishes: update the State Dashboard and link the new document back to the briefing/project that requested it. File-based vaults use `[[wikilinks]]`; Notion uses Relation properties.

---

## Sub-flow routing

| If the user's request is about… | Hand off to |
|---|---|
| Adding/managing tasks (add a task, mark done, what's on my plate, what's blocked, move task to project) | `flows/task-management.md` |
| Smart nesting / a new domain or department / dangling records / file cabinet structure | `flows/file-cabinet.md` |
| Setting up watchers / Phase 1 / Express Lane | `flows/watcher-setup.md` |
| Captain's Log recall ("what did I log about X", "show me my wins", "what did I decide about Y") | `flows/captains-log-recall.md` |
| A quick documentation phrase ("log this", "document this", "captain's log", "remember this", "journal that", etc.) | `flows/documentation-routing.md` |

If the request fits one of those rows, hand off and stop reading this file.

---

## Inline handling (handled here — these are short)

### Status questions
Answer from State Dashboard + Live Feed + messaging + Tasks database. Link to relevant files.

### Watch list
"Keep an eye on X" → snapshot first (G2), then add to `State Dashboard → Watch List`. Confirm in plain English.

### Priority changes
Snapshot first (G2), then update State Dashboard immediately. Confirm.

### Missed messages
"You missed an email from X" → snapshot first (G2), then add to `State Dashboard → Missed Messages` (sender pattern + reason).

### Connector updates
"I connected [tool]" or "update my connectors" → re-run connector detection (same as setup S3) and update the Connectors table in the State Dashboard. Snapshot first (G2).

### Anything else
Use judgment. Apply Universal Gates. When in doubt, ask.

---

## Proactive documentation (during conversation)

While the user talks, watch for moments that should be captured.

- **Operational updates** (task finished, status changed, new blocker, priority shift) → update the relevant Task, Project, or State Dashboard record automatically and explain: "That changes the status of [project] — I've updated the project record and noted [detail]." Snapshot first (G2) for State Dashboard writes.
- **Captain's Log moments** (ideas, reflections, lessons, personal wins, things worth remembering that don't belong to a specific task) → **never auto-write.** Explain why it seems like a Captain's Log entry: "That insight about [topic] sounds like something worth keeping in your Captain's Log as a Learning — it's not tied to a task, but you might want to recall how you arrived at that thinking. Want me to log it?" If the user declines, move on.
- **Judgment calls** → offer once, explain your thinking: "That sounds like it could be a new watch list item — want me to add it?" If declined, move on. Don't offer more than twice per session to avoid nagging.

Detailed routing logic for documentation phrases is in `flows/documentation-routing.md`.

---

## End-of-session documentation sweep

When the user signals they're wrapping up:

1. Quickly review what happened in the session.
2. Check for undocumented decisions, completed tasks, status changes, noteworthy moments.
3. Apply any obvious operational updates automatically (snapshot first for State Dashboard) — list what you did.
4. For judgment calls, offer once briefly: "Before you go — [item] might be worth a log entry. Want it?"

**Abrupt-exit safety net.** If the user says "gotta go" or stops responding, save the State Dashboard update above all else. Even a partial update beats none. Skip the sweep — State Dashboard save comes first.

---

## Session discipline

- Keep sessions short: a few tasks, then end.
- Files are the memory, not the chat. All important info lives in the documents.
- Never take action without approval (G1). Alerts are the one exception.
- Always use the standard, documented way to do things — no hacks, shortcuts, or workarounds. If the standard way is unknown, read the documentation first. If still unclear, ask.
