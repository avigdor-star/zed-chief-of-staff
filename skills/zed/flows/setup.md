# Flow: Setup

**What this flow does:** First-time setup walkthrough — creates the Chief of Staff structure, populates the State Dashboard, runs connector tests, sets up Captain's Log, prompts for encrypted backup. Runs S1 through S7 in order.

**When to load it:** Bootstrap can't find a Chief of Staff location, OR `setup_status` is `incomplete`, OR the user explicitly says "run setup."

**Prerequisites:** Bootstrap complete (or partially complete with no CoS location found). Universal Gates G1-G8 from SKILL.md apply throughout.

---

## Resume-from-partial-setup rule

Before starting S1, check the State Dashboard (if one exists) for pre-filled fields. Skip any sub-step whose answer is already recorded:
- If `My Setup → Name / Email / Role / Company / Chief Name / Alert Threshold` are all filled → skip S2.
- If `My Setup → Platform` is filled → skip S1.5 (platform choice).
- If `System Health → Connectors` table has values other than the default → skip S3 for those connectors.
- If the folder/database structure already exists per the chosen platform → skip S4.
- If `My Setup → setup_status` is `complete` or `complete-with-warning` → only run S6 (backup) if its status is still unresolved.

Tell the user at the top: "Looks like setup was partially completed. I'll pick up from [where we left off] instead of starting over. Correct me if I got that wrong." Wait for confirmation before proceeding.

---

## S1 — Introduce the system

Tell the user:
> "This system works like a personal knowledge area — a structured set of notes that track your priorities, projects, and status over time. Each note connects to others, so your Chief of Staff can follow the thread. You'll pick which app or service holds these notes in the next step. All four supported options are free."

---

## S1.5 — Platform choice

Load `references/platform-choice.md` and run the flow. It asks 1–3 short questions and lands the user on one of these four platforms:
- `obsidian`
- `logseq`
- `markdown-folder`
- `notion`

Store the pick (lowercase, exactly one of the four values above). It's used in S4 (structure), S5 (State Dashboard Platform field), and S6 (backup path).

After the pick, briefly confirm in plain English:
- Notion → "Okay — Notion it is. I'll create a Chief of Staff parent page in your Notion workspace with databases for briefings, projects, people, and so on. That happens in a minute — first I need a few more details from you."
- Obsidian → "Okay — Obsidian it is. I'll create a Chief-of-Staff folder in the folder you point me at. First, a few more details."

Then load the right conventions file for the rest of setup:
- `obsidian`, `logseq`, or `markdown-folder` → load `references/file-based-conventions.md`
- `notion` → load `references/notion-conventions.md`

---

## S2 — Setup questions

Ask in plain language, one group at a time.

**About you:**
- What's your name?
- What's your email address?
- What's your role? (e.g., founder, ops manager, team lead)
- What company or team do you work for?
- What do you want to call your Chief of Staff? Pick a name — you'll use it to summon it. (e.g., "Zed", "Friday", "Jarvis", or just "Chief".)
- What personality should your Chief of Staff have?
  - **"Professional"** (default) — clear, direct, no frills
  - **"Playful & lighthearted"** — fun energy, gentle humor, still tactful
  - **"Dry wit"** — understated humor, a bit sarcastic
  - **"Warm & encouraging"** — supportive, upbeat
  - Or describe something else entirely — it's your Chief.
  Store the answer in `State Dashboard → My Setup → Personality`.

**Logseq sub-question (only if platform = `logseq`):**
- "Logseq can store pages as plain folders (like Obsidian) OR as namespaces (Logseq's native style, using `___` as a separator). Which are you using? If you're not sure, say 'plain folders' — that's the most common and closest to Obsidian." Record in `State Dashboard → My Setup → Logseq Placement` as either `folders` or `namespaces`. See `references/file-based-conventions.md` → "Platform Differences — Logseq" for what each means.

**Your domains:**
- What are the big areas of your life this system should cover? These are your "domains" — the top level. For most people, this is their businesses or jobs plus personal life. (e.g., "My Agency", "My Course Business", "Personal", "Shared/Admin".) Most people have 3–5 domains.

**Your departments:**
- Within each domain, what are the functional areas? (e.g., under a business: Marketing, Product, Sales, Finance. Under Personal: Health, Family, Home Maintenance, Vehicle.) Don't overthink it — you can add more later.

**Your projects:**
- What are you currently working on? (List active projects or areas of focus. For each one, note which department it falls under.)

**Your priorities:**
- What are your top 3 priorities right now?
- What does a successful week look like for you?

**Key people:**
- Who are the people whose messages you should never miss? (Names and why they matter.)

**Alert preferences:**
- How aggressive should alerts be?
  - **"Only emergencies"** — payment failures, system outages, hard deadlines due TODAY
  - **"Important stuff"** — the above, plus someone waiting 24+ hours for your reply, or a key person reaching out
  - **"Everything notable"** — all of the above, plus anything that needs a decision from you

---

## S3 — Detect available connectors

Test which tools work by trying a simple read from each:
- Email: try reading recent messages. Note Gmail or Outlook.
- Calendar: try reading today's events. Note Google Calendar or Outlook.
- Task tracker: try reading tasks. Note which tool (Airtable, Notion, Linear, Asana).
- Messaging: try reading a channel. Note Slack or Teams.

**Email send test (required if email is connected).** Reading mail isn't enough — the 3-tier alert cascade depends on being able to SEND.

1. **Prefer the connected account's address, not the typed one.** If the email connector exposes the authenticated user's address, use that as the target and skip step 2.
2. **If the connected account doesn't expose its own address**, re-confirm: "I'm about to send a test alert to `[email from S2]`. Is that the correct address? (yes / no — let me fix it)." Do NOT send until the user confirms.
3. Send a short test email: Subject "CoS setup test", body "If you're seeing this, your Chief of Staff can send alerts. Reply 'got it' or just confirm in chat."
4. Ask: "Did the test email arrive? (yes / no / didn't check)." If yes → mark email `send: working`. If no or didn't check → mark email `send: unverified` and note in the Connectors table that alerts may fall through to messaging / Live Feed only.

If a tool isn't connected or the test fails, mark "Not connected" — don't ask the user to set it up right now. Tell them what you found: "I can see your email and calendar. I don't have access to a task tracker or messaging tool yet — that's fine, we can add those later."

---

## S4 — Create the Chief of Staff structure

Branch on the platform chosen in S1.5.

### If platform = `obsidian`, `logseq`, or `markdown-folder`

Create the `Chief-of-Staff/` folder in the user's vault with:

```
Chief-of-Staff/
├── _index.md                ← One-screen map; write this FIRST (template in file-based-conventions.md)
├── State Dashboard.md
├── Live Feed.md
├── Domains/                 ← One file per life domain
├── Departments/             ← One file per department
├── Projects/                ← One subfolder per active project
├── Tasks/                   ← One file per task
├── Briefings/
├── Research/
├── Drafts/
├── Action-Plans/
├── People/
├── Reference/
└── Archive/
```

Write `_index.md` first — it's the contract every future session reads. **Guard:** if `_index.md` already exists, do NOT overwrite. Read the existing file; if it looks valid (has `type: index` in frontmatter and a Folders section), leave it alone. If it's malformed: "An `_index.md` already exists but looks incomplete. Overwrite with the standard template, or leave it alone for you to fix? (overwrite / leave / show me)."

### If platform = `notion`

Use the Notion connector to create under a single parent page named **Chief of Staff**:

```
Chief of Staff (parent page)
├── State Dashboard (page)
├── Live Feed (page)
├── Snapshots (page)              ← page-snapshots live here
├── Domains (database)
├── Departments (database)
├── Projects (database)
├── Tasks (database)
├── Briefings (database)
├── Research (database)
├── Drafts (database)
├── Action Plans (database)
├── People (database)
└── Reference (database)
```

For each database, create properties and templates per `references/notion-conventions.md`. Lay out the State Dashboard and Live Feed per the sections in that file.

After creating databases, populate Domains and Departments with the entries from S2. For each domain, create one record. For each department, create one record and link to its domain. Leave Projects and Tasks empty.

> **Note:** `Chief-of-Staff/State Dashboard.md` is NOT a copy-paste template for the Notion State Dashboard page. The Notion page is built from Notion blocks — headings, callouts, two-column tables, embedded linked-database views — not from markdown. Use the schema in `notion-conventions.md` as the source of truth for the Notion layout.

> **Expected after setup:** all eleven databases will be mostly empty — Domains and Departments are pre-populated from S2, everything else fills up as the system runs. Reassure the user of this before handing off from S4.

---

## S4.5 — Create the Captain's Log (mandatory)

Captain's Log is mandatory on every platform. Load `references/captains-log.md` for the schema and platform-specific layout.

Create the structure:

- **`notion`** → create a `Captain's Log` database at the **workspace top level** (NOT under the Chief of Staff parent — kept independent so the user can find it). Apply the schema; add the three default views (Recent, By Type, By Project).
- **`obsidian`** / **`markdown-folder`** → create the folder `Chief-of-Staff/Captain's Log/` (empty, ready for entries).
- **`logseq`** → don't create the page proactively. Record in the State Dashboard that Captain's Log lives at the page `Captain's Log` (or use the namespace path if `namespaces` placement). The page is created on the first entry.

**Then write the one auto-generated first entry** to mark today's setup. This is the ONE entry the system creates on the user's behalf:

- Title: `Chief of Staff set up`
- Date: today
- Type: `Milestone`
- Project: `Chief of Staff`
- Details: short summary — platform chosen, date, anything notable

Tell the user where Captain's Log lives and how to add entries:

> "Captain's Log is set up. To log something, just say 'log this: …' or 'captain's log: …' and I'll figure out where it belongs — task update, project note, or the Captain's Log — and write it there. I already added one entry to mark today's setup."

---

## S5 — Populate the State Dashboard

Fill in the State Dashboard from S2 answers, following the conventions file for the chosen platform.

Fill in:
- **My Setup** (name, email, role, company, alert threshold, chief name, personality, **platform — must match the pick from S1.5**, `setup_status` — set to `complete` at end of S6, or `complete-with-warning` if backup was deferred)
- **Active Projects** (from project list)
- **High-Priority People** (from key people list)
- **This Week's Focus** (from priorities)
- **Connectors table** (from S3)
- **Automation Status** — set the **Phase 0 / Week 1** row's Status cell to `Active`. All other rows stay at `Not started`.
- **Check-in State** (frontmatter `rollout_reminder:` block) — leave most fields at defaults. Only set `next_eligible` to today + 7 days so the first rollout nudge is eligible a week from today.
- **Watch List** → empty, note: "Tell me to 'keep an eye on' something and it goes here."
- **Open Decisions** → empty, note: "Decisions needing your input will appear here."
- **Missed Messages** → empty (will fill as the system learns)
- **Session anchor (platform-aware):**
  - `obsidian` / `logseq` / `markdown-folder`: set `last_updated:` in the State Dashboard's frontmatter to today's date.
  - `notion`: add a `Last Session At` date property to the State Dashboard page (NOT the built-in `Last edited`, which would update on every agent edit and make the anchor unreliable) and set it to today.

---

## S6 — Encrypted backup (strongly recommended)

Encrypted backup is strongly recommended before completing setup. A knowledge base with no encrypted backup will leak private data if the machine is stolen or the Notion account is lost, and will be lost entirely if the disk dies or Notion disappears. If the user defers, setup is marked `complete-with-warning` and Axiom 1 re-surfaces the backup flag at every briefing until resolved.

Adapt the prompt to the platform.

**For `obsidian` / `logseq` / `markdown-folder`:**

> "One last step. Your Chief of Staff area will hold your priorities, client notes, drafts — things you wouldn't want anyone else to read. Before we finish setup, we need to turn on two things: (1) FileVault, so a stolen laptop is unreadable, and (2) an end-to-end encrypted off-machine backup, so if the laptop dies your notes survive. Both are free. Should I walk you through it now?"

**For `notion`:**

> "One last step. Your Chief of Staff area will live in Notion. Notion encrypts data on its servers, but Notion's staff could technically read it — it's not end-to-end encrypted. To make sure your notes survive a Notion outage, account loss, or worst-case data exposure, we'll set up a weekly export that lands in an encrypted folder on your computer. It's free and takes about 10 minutes. Should I walk you through it now?"

If the user says yes → load `references/backup-setup.md` and walk them through it. The file's Step 1 branches on platform: file-based users go through Paths A–D; Notion users go to Path E. When finished, set `State Dashboard → My Setup → setup_status` to `complete`.

If the user says "later" → set `setup_status` to `complete-with-warning`, set `System Health → Connectors → Backup → Status: NOT_CONFIGURED`, and re-prompt at every briefing until done (Axiom 1).

---

## S7 — Finish setup

Tell the user: "You're all set. Your Chief of Staff is named [name]. Say 'briefing', or say [name] directly, whenever you want your first update."
