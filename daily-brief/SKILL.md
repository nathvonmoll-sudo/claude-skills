---
name: daily-brief
description: >
  Check the user's calendar and recent emails to produce a clear, actionable
  brief of what's planned and what needs attention — for today, the next few
  days, next week, or any specific day. Also scans emails for meetings,
  deadlines, or commitments that aren't yet on the calendar and offers to add
  them. ALWAYS use this skill when the user asks: "what's on today", "plan my
  day", "what's coming up", "brief me for the week", "what do I have on
  [day]", "what do I need to do today", "anything I'm missing", "check my
  schedule", "what's planned", or any similar request for a daily or weekly
  overview. Requires Google Calendar and Gmail to be connected.
---

# Daily Brief Skill

Pulls calendar events and scans recent emails to produce an actionable brief,
then offers to add any email-derived events that are missing from the calendar.

If the user has Microsoft 365 connected instead of Google, use the outlook_email_search and outlook_calendar_search tools from that connector in place of the Gmail and Google Calendar tools above. The logic is identical.

---

## Step 1: Determine the time window

Read the user's request and map it to one of these windows:

| Request | Window |
|---|---|
| "today" / "plan my day" / "what's on" | Today only |
| "tomorrow" | Tomorrow only |
| "next few days" / "coming up" | Today + next 3 days |
| "this week" / "week ahead" | Today through end of this working week (Friday) |
| "next week" | Monday–Friday of next week |
| "specific day" (e.g. "Wednesday", "15 June") | That day only |

If unclear, default to "today + next 3 days".

---

## Step 2: List calendars

Use `Google Calendar: list_calendars` to get all available calendars and their
IDs. Note which ones are active. If the user has multiple calendars (personal,
work, shared), pull events from all of them.

---

## Step 3: Pull calendar events

Use `Google Calendar: list_events` for each active calendar, scoped to the
time window. For each event collect:
- Title
- Date and time (or all-day flag)
- Duration
- Location or video link (if present)
- Description / notes (if present)
- Attendees (if it's a meeting)
- Which calendar it came from

Sort all events chronologically across calendars.

---

## Step 4: Scan recent emails for missing events

Use `Gmail: search_threads` to pull relevant recent emails. Use these queries
to target the most useful signals:

```
is:unread newer_than:3d
```

Then also run a targeted search for scheduling language:

```
newer_than:5d (meeting OR call OR deadline OR "by Friday" OR "by Monday" OR
"let's connect" OR "can we meet" OR "scheduled for" OR "confirmed for" OR
invite OR appointment)
```

Fetch the plaintext body of any threads that look relevant using
`Gmail: get_thread`.

For each email, look for:

**Meeting/event signals:**
- A specific date and time mentioned for a call, meeting, or appointment
- A calendar invite link or "accepted" / "confirmed" language
- "Let's meet on...", "Can we do...", "I've booked..."

**Deadline/commitment signals:**
- "Please send by...", "Due by...", "I need this by..."
- "I'll have it to you by...", "We need to decide by..."
- A task the user has committed to delivering on a specific date

**For each signal found, check:** Is there already a matching event on the
calendar? Match by date + rough topic. If yes, skip it. If no, flag it as
a missing event.

---

## Step 5: Build the brief

Output the brief in clean, scannable prose. Do not use excessive headers or
turn it into a wall of bullets. Structure:

### [Time window title — e.g. "Today — Thursday 25 June" or "Week of 23–27 June"]

**Morning / Afternoon / Evening** (group events by time of day for single-day
briefs; for multi-day briefs, group by day)

For each event:
- Time + title (bold the title)
- One line of context if useful (location, who it's with, what it's about)
- Skip redundant detail

**From your emails — things not yet on the calendar:**
List any email-derived events or deadlines found in Step 4 that aren't
already on the calendar. For each one:
- What it is and when
- Which email it came from (sender + subject line)
- Whether it looks like a confirmed commitment or just a suggested time

End with a short "anything else" note if there are unread emails that look
time-sensitive but don't fit neatly into the above.

---

## Step 6: Offer to add missing events to the calendar

After presenting the brief, if any missing events were found in Step 4, ask:

> "I spotted [N] thing(s) from your emails that aren't on your calendar yet.
> Want me to add them?"

If the user says yes (for all or specific ones):

For each event to add, confirm the details before creating:
- Title (inferred from email context)
- Date and time (extracted from email — if ambiguous, ask)
- Duration (default 30 min for calls, 60 min for meetings, unless specified)
- Which calendar to add it to (default: primary)

Then use `Google Calendar: create_event` to create it.

Confirm each one after creation: "Added: **[title]** on [date] at [time]."

---

## Tone and format notes

- Keep it conversational and concise — the user should be able to scan this
  in 30 seconds
- Don't pad with filler ("Here's what I found for you!")
- If the calendar is empty for the window, say so directly and move straight
  to the email scan
- If there's nothing in emails either, say that clearly rather than
  inventing structure
- For multi-day briefs, one short paragraph per day is usually enough unless
  a day is packed
- Flag anything that looks like a conflict (two events at the same time)
  without being asked

---

## Required connectors

- **Google Calendar** — for reading events and creating new ones
- **Gmail** — for scanning emails

If either is not connected, tell the user which one is missing and what it's
needed for, then proceed with whatever is available.

