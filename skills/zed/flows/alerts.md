# Flow: Alerts (Step 3 cascade)

**What this flow does:** Sends out-of-band alerts when conditions match, using a 3-tier cascade (email → messaging DM → Live Feed). Auto-fires — alerts are the one exception to the approval gate (G1).

**When to load it:** Auto-triggered by alert conditions detected during a briefing or interactive session. Not user-invoked.

**Prerequisites:** Bootstrap complete. Universal Gates G1-G8 apply, with G1 explicitly waived for alerts (they auto-fire by definition). Alert threshold respected per `State Dashboard → My Setup → Alert Threshold`.

---

## Alert trigger conditions

Fire an alert ONLY when one of these is true:
- Someone is waiting **24+ hours** for the user's reply
- A hard deadline is **within 12 hours** with no activity
- A **High-Priority Person** (from State Dashboard) has reached out
- A **payment, billing, or financial issue**
- A **connector failure** is blocking the briefing

If none of these match, do not fire an alert.

---

## Alert threshold (respect the user's setting)

Read `State Dashboard → My Setup → Alert Threshold` and filter the trigger list:

- **"Only emergencies"** → payment failures, system outages, hard deadlines TODAY only.
- **"Important stuff"** → emergencies + people waiting 24+ hours + high-priority person reaching out.
- **"Everything notable"** → all of the above + any new item that needs a decision from the user.

---

## The 3-tier cascade

Use ALL available tiers that are currently working. Live Feed is the always-on fallback — even if email and messaging both fail, the alert still lands there and surfaces at the top of the next briefing.

### Tier 1 — Email

- Subject: `CoS Alert: [summary]`
- Body: 3–4 sentences. What happened, why urgent, what the user might do.
- Send to the user's email (from `State Dashboard → My Setup → Email`).

**Skip Tier 1 if** `State Dashboard → System Health → Alert Email Status` is "Draft only — using messaging DM + Live Feed fallback" (i.e., the setup S3 email send test failed or was unverified). In that case, save the alert text as a draft in the user's email so they can review/send manually, and proceed straight to Tier 2.

### Tier 2 — Messaging DM

- Same content as Tier 1, formatted for the platform (Slack DM or Teams DM).
- Send to the user's DM channel.

If messaging is not connected, skip Tier 2 and proceed to Tier 3.

### Tier 3 — Live Feed ALERTS section

Always fires. Append an entry to:
- File-based: `Chief-of-Staff/Live Feed.md` under the `## ALERTS` section.
- Notion: the Live Feed page's `Urgent Alerts` callout.

Format:
- Timestamp
- Summary
- Why urgent
- Status: UNREAD

This entry will be the first thing surfaced in the next briefing's Alerts section.

---

## After the cascade

Log the alert in working memory for the session — don't fire the same alert twice within one session even if the trigger condition still matches.

If the alert covers something the user already acknowledged in conversation (e.g., they said "I know about that, I'm handling it"), don't fire — set the Live Feed entry's Status to HANDLED instead.
