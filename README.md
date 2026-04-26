# Zed — AI Chief of Staff

From-scratch rebuild of the `chief-of-staff` skill. Same features, cleaner architecture: a thin `SKILL.md` router plus on-demand flow files.

## What it does

- Section-by-section briefings ("briefing", "what's going on", "catch me up")
- Documentation routing ("log this", "captain's log") — operational record first, Captain's Log as fallback
- Task management ("add a task", "what's on my plate", "what's blocked")
- Smart nesting (Domain → Department → Project → Task)
- Phased rollout of automatic background scans (watchers)
- Captain's Log recall ("what did I log about X")
- Auto-fired alerts via 3-tier cascade (email → messaging → Live Feed)

## Supported platforms

Pick one during setup: Obsidian, Logseq, plain markdown folder, or Notion.

## Coexistence with chief-of-staff

If you have the older `chief-of-staff` plugin installed, pause it in the plugin manager before testing Zed. Both skills read the same data — switching between them is just switching brains, not data.
