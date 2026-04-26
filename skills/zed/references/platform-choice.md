# Platform Choice — Guided Flow

> Run this during first-time setup (SKILL.md Step S1.5) to land the user on one of four supported platforms. If the user already has a platform picked, skip this file and just record their pick in `State Dashboard → My Setup → platform:`.

## The four supported platforms

Only these four. No others.

1. **Obsidian** — local markdown files, free, fully encryptable, great desktop + mobile apps.
2. **Plain markdown folder** — just a folder of `.md` files, viewed with any app the user likes (or a text editor). Free, fully encryptable, bare minimum setup.
3. **Logseq** — open-source, file-based, outliner-style. Same file model as Obsidian for the AI.
4. **Notion** — cloud-based, polished mobile UX, shared with the team easily. Encrypted at rest but NOT end-to-end on free plans (live content sits on Notion's servers; encrypted backup is handled via weekly export).

Apple Notes, Google Docs, Microsoft tools, and Airtable-as-primary are explicitly **not** supported. Do not offer them.

---

## Opening the conversation

Say this to the user, in these words or close to them:

> "We need to pick where your Chief of Staff content will live. There are four options that all work well — Notion, Obsidian, Logseq, or a plain folder of notes. Want me to walk you through the trade-offs first, or just ask you a few quick questions and land on one?"

- **"Walk me through it"** → load the Trade-off walkthrough below, then start the questions.
- **"Just ask me the questions"** → skip straight to Q1.

---

## Trade-off walkthrough (only if the user asks)

Read these four paragraphs to the user in plain English. Keep it short.

**Notion.** Cloud-based. Works beautifully on your phone. Polished, clickable, looks like a nice app. Downside: Notion's servers can see your live content — it's encrypted in storage but not end-to-end. If the content is sensitive, we back it up weekly to an encrypted folder on your machine. Best for people who want the most polished phone experience and aren't putting their deepest secrets in here.

**Obsidian.** Free app you install on your computer (and phone, if you want). Your notes are regular files on your machine, and Obsidian gives you a clickable, visual interface on top of them — links between notes, a graph view, search. Because the files live on your machine, you can wrap them in end-to-end encryption easily. Best for people who want a clickable setup and care about privacy.

**Logseq.** Similar to Obsidian — free, installs on your computer, files live on your machine, fully encryptable. The main difference is the writing style: Logseq organizes everything as bullet outlines rather than flowing notes. Best for people who already think in outlines or already use Logseq.

**Plain markdown folder.** No new app to learn. Your vault is just a folder of `.md` text files — you can open them with any text editor, any markdown viewer, or the Mac's built-in TextEdit. Fully encryptable. Best for people who want the simplest possible setup and don't need a fancy interface.

After the walkthrough, say: "Want me to ask you the questions now to land on one?"

---

## The questions

Ask these in order. If the user answers "don't know" or "no preference" on any question, follow the arrow to the default.

**If the user already said during the trade-off walkthrough that they want a specific one, skip the questions and use their pick.**

---

### Q1. "Are you already using Notion, Obsidian, or Logseq for something else?"

- **Notion** → recommend Notion. Done.
- **Obsidian** → recommend Obsidian. Done.
- **Logseq** → recommend Logseq. Done.
- **None of those / I don't know** → go to Q2.

### Q2. "Where will you mostly use this — on your phone, mostly on your computer, or about equal?"

- **Phone-heavy** → go to Q3.
- **Computer-mostly** → go to Q4.
- **Equal / not sure** → recommend **Notion**. Done.

### Q3 (phone-heavy users). "How sensitive is what you'll put in here — private client details and personal thoughts, or mostly regular notes?"

- **Sensitive** → recommend **Obsidian**. (Obsidian has a solid free phone app and stays inside your encrypted backup. Notion's free plan isn't end-to-end encrypted, so sensitive content is a bad fit there.)
- **Regular notes / I don't know** → recommend **Notion**. Done.

### Q4 (computer-mostly users). "Do you want a clickable, visual setup where notes link to each other — the simplest possible thing with no new app to learn — or you're not sure?"

- **Clickable / visual** → recommend **Obsidian**. Done.
- **Simplest possible** → recommend **Plain markdown folder**. Done.
- **Not sure** → recommend **Notion**. Done.

---

## After the pick

1. **Confirm the pick in plain English.** Example: "Okay — we're going with Notion. That gives you the polished phone experience. I'll walk you through the Notion-specific setup next."

2. **Write the pick to the State Dashboard.** In `State Dashboard → My Setup`, set:
   ```
   | **Platform** | notion |
   ```
   Value is one of: `obsidian`, `markdown-folder`, `logseq`, `notion` (lowercase).

3. **Load the right conventions file next:**
   - `obsidian`, `markdown-folder`, or `logseq` → load `references/file-based-conventions.md` (covers all three).
   - `notion` → load `references/notion-conventions.md`.

4. **If the pick is `logseq`, ask the Logseq placement sub-question** in S2: "Logseq can store pages as plain folders (like Obsidian) OR as namespaces (Logseq's native style, using `___` as a separator). Which are you using? If you're not sure, say 'plain folders' — that's the most common and closest to Obsidian." Record the answer in `State Dashboard → My Setup → Logseq Placement` as either `folders` or `namespaces`. See `references/file-based-conventions.md` → "Platform Differences — Logseq" → "Page placement" for what each means and how briefing paths differ.

5. **Remember: encrypted backup is strongly recommended (Axiom 1).** The backup path depends on the platform (see `references/backup-setup.md`). If the user defers, setup is marked `complete-with-warning` and Axiom 1 re-surfaces the flag at every briefing until resolved.

---

## Default summary

If the user picks "don't know" on every single question, they land on **Notion**. That's the intentional default — it's the easiest on-ramp for a non-technical user who has no strong opinions. Every other platform requires an explicit signal to be chosen.

## 60-second test

If a non-technical user can't reach a pick from this flow in under 60 seconds, the flow has failed. If that happens, simplify — don't add more questions.
