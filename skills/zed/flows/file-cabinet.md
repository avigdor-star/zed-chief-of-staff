# Flow: File Cabinet (Smart Nesting)

**What this flow does:** Infers the right level (Domain / Department / Project / Task) for something the user mentions, and offers to nest it properly. Also handles dangling records surfaced in the briefing's File Cabinet Check.

**When to load it:** User mentions a new task, project, department, or domain that isn't already nested correctly. Also loaded by `flows/briefing.md` for the File Cabinet Check section.

**Prerequisites:** Bootstrap complete. Universal Gates G1-G8 from SKILL.md apply — especially G1 (approval).

---

## The four levels (smallest to largest)

- **Task** — a specific action. Signals: "I need to …", "Remind me to …", "Schedule …", "Add …". One thing, one sitting. Example: "buy diapers."
- **Project** — a specific initiative made of multiple tasks. Signals: "I want to build/create/launch/set up …", "Let's kick off …". Weeks to months of work. Example: "car maintenance system."
- **Department** — a functional area within a life domain. Signals: naming a category of ongoing work, not a one-time initiative. Permanent. Example: "procurement."
- **Domain** — a whole life entity. Signals: naming a business, a big life area, a shared/admin bucket. Top-level. Example: "Family."

---

## Inference rules

When the user mentions something new, classify by the signals above. If genuinely ambiguous, ask: "Sounds like that could be a Project (multi-step initiative) or a Task (single action). Which?"

---

## Offer to nest — don't force

Once the level is inferred, check whether parents exist in the file cabinet:

- **All parents exist:** propose the home and confirm. "Sounds like a task — I'd file it under Personal → Home → Home Maintenance. Sound right?"
- **Parents missing:** offer to create the missing chain. "Sounds like a task ('buy diapers'). No Family domain yet — want me to set up Family → Procurement → Hygiene first, then add the task? Or skip the nesting and leave the task on its own?"
- **User skips nesting:** create the record with no parents. It still works, still surfaces in briefings. Not a violation — an acceptable state.

Apply the same pattern when the user creates a project, department, or domain. Infer, propose, offer to create missing parents, never force.

---

## Dangling records

A dangling record (task without a project, project without a department, department without a domain) is acceptable. It's a nudge target on the next brief, not an error.

The briefing's **File Cabinet Check** section surfaces dangling records as a soft optional nudge. The user can home them then or leave for later. Never frame as a violation.

---

## Where things get created

**File-based (`obsidian` / `logseq` / `markdown-folder`):**
- Domain → `Chief-of-Staff/Domains/[domain-slug].md` with `type: domain` frontmatter.
- Department → `Chief-of-Staff/Departments/[department-slug].md` with `type: department` frontmatter linking to its domain.
- Project → `Chief-of-Staff/Projects/[project-slug]/` folder. Add an index file inside with `type: project` frontmatter linking to its department.
- Task → `Chief-of-Staff/Tasks/[task-slug].md` with `type: task` frontmatter linking to its project.

**Notion:**
- Domain → new row in the Domains database.
- Department → new row in the Departments database, with `Domain` relation set.
- Project → new row in the Projects database, with `Department` relation set.
- Task → new row in the Tasks database, with `Project` relation set.

Full schemas in `references/file-based-conventions.md` and `references/notion-conventions.md`.

---

## Snapshot rule

The hierarchy databases (Domains / Departments / Projects / Tasks) are NOT part of the State Dashboard. Snapshot gate G2 does NOT apply to creating or editing those records. It DOES apply if the user's request also touches State Dashboard content (e.g., updating Active Projects in the dashboard after creating a new project).
