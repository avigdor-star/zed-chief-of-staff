# Signal Filters & Priority Ranking

## User-Defined Filter Rules (highest priority — checked first)

The State Dashboard contains a **Filter Rules** section where the user lists senders, domains,
companies, and topics they NEVER want to see. These override everything else in this file.

**Before applying any other filter logic, check every candidate item against the Filter Rules
list.** If an item matches a filter rule — by sender name, email domain, company name, site
name, or topic — drop it immediately. Do not surface it in any briefing section, do not carry
it over from a previous briefing, do not include it in alerts. A filter rule added after an
item was first surfaced still kills that item on the next pass.

This check applies to ALL briefing sections: Top 3, Messages That Need You, Carryover,
Recommended Actions, Risks/Blockers, and Watch List Check.

## Always Surface

- Deadlines within 48 hours
- Unread emails from real people (not newsletters, not automated notifications)
- Calendar conflicts or back-to-back meetings with no prep time
- Tasks marked urgent or high priority in the task tracker
- Items flagged as risks or blockers in the previous briefing
- Items on the Watch List in the State Dashboard
- Anything from a person on the High-Priority People list in the State Dashboard
- Messages matching patterns in the Missed Messages section (senders or subjects previously missed)

## Skip Unless Asked

- Marketing newsletters and promotional emails
- Automated tool notifications (GitHub bots, CI alerts, Airtable alerts, etc.)
- Tasks with no due date untouched for 2+ weeks (surface in weekly review only)
- Routine recurring calendar events (unless there's a conflict)

## Priority Ranking

When ranking importance, use this order:

1. **Hard deadlines** — things expiring within 48 hours
2. **People waiting** — someone is blocked until the user responds
3. **Revenue impact** — directly tied to sales, launches, or client delivery
4. **Strategic work** — moves the business forward but not time-sensitive
5. **Everything else**

If unsure whether something is urgent, ask the user. Do not guess.

## Email Filtering Details

**REPORT if:**
- Email is from a real person (not a newsletter, not automated)
- Email asks a question or requests action
- Email is from someone on the High-Priority People list
- Email mentions a deadline, payment, or urgent matter
- Email matches a pattern from the Missed Messages section

**SKIP if:**
- Newsletters, marketing emails, promotional content
- Automated notifications from tools (GitHub, Airtable, Supabase, CI/CD, etc.)
- Emails that are FYI-only with no action needed

## Deduplication

Before writing any item to the Live Feed or posting to a messaging channel, check for
existing entries with the same sender + subject (for emails) or same task name (for task
tracker items) that are still marked NEW or CARRIED OVER. Don't duplicate them.
