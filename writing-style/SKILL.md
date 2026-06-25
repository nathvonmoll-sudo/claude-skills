---
name: writing-style
description: >
  Capture and apply a user's personal writing style based on their sent emails,
  then use it to write emails, messages, and other content that sounds exactly
  like them. ALWAYS use this skill when the user asks to "write in my style",
  "write this like I would", "write an email for me", "sound like me", "capture
  my writing style", "set up my style", "update my style", or wants any written
  communication that matches their voice. Also trigger when the user wants to
  draft a professional email, reply, or message and hasn't specified a different
  style — default to checking for a saved style profile first. Two modes: SETUP
  (reads sent emails via Gmail, creates and saves a style profile) and WRITE
  (loads the saved profile and applies it to draft content).
---

# Writing Style Skill

This skill has two modes. Read the user's intent and jump to the right one.

---

## MODE 1: SETUP — Capture the user's style

**When to run this mode:** The user says something like "capture my writing style", "set up my style", "update my style from my emails", or they haven't run setup before and want to write in their style.

### Step 1: Read sent emails via Gmail

Use the Gmail `search_threads` tool to retrieve the 30 most recent sent emails:
```
query: "in:sent"
pageSize: 30
view: THREAD_VIEW_MINIMAL
```

Then use `get_thread` with `messageFormat: FULL_CONTENT` on a sample of 8–12 of the most substantive threads (skip threads with only one-word/auto replies, and threads where the user is clearly just forwarding something). Aim for variety across: cold outreach, professional working relationship emails, personal/close contact emails, and quick replies.

### Step 2: Analyse the style

Read through the plaintext bodies of the user's sent messages only (not replies from others). Extract the following:

**1. Greeting patterns**
- How does the user open emails to people they don't know? (e.g. "Good morning." / "Good day, [Name].")
- To working contacts? (e.g. "Hi [Name],")
- To close contacts? (e.g. "Hey [Name]!")
- Do they follow the greeting with a standard courtesy line? (e.g. "I hope you are doing well.")

**2. Tone register by relationship**
- Map out 2–4 relationship tiers the user writes to, and describe the tone shift between them
- Note: formal vs semi-formal vs casual markers

**3. Sentence and paragraph structure**
- Average sentence length (short/medium/long)
- Do they front-load information or build up to it?
- How do they handle requests — direct ask, or context-first then ask?
- Do they use numbered lists, bullet points, or prose for multi-part content?

**4. Sign-off patterns**
- Closing phrase(s) used (e.g. "Kind regards," / "Thanks," / "Thank you!")
- Name format (first name only, full name, with/without period)

**5. Distinctive vocabulary and phrasing**
- Recurring phrases or sentence starters unique to their voice
- Words they seem to avoid (corporate jargon, filler phrases, etc.)
- Any notable patterns in how they express appreciation, explain technical things, make requests, or close a loop

**6. What they never do**
- Note absent patterns: no hollow filler phrases like "Please don't hesitate to reach out", no sycophantic openers, no over-hedging, etc.

**7. Formatting habits**
- Use of whitespace, paragraph breaks, numbered/unnumbered lists
- Email length by context (brief reply vs. substantive outreach)

### Step 3: Write and save the style profile

Generate a style guide as a markdown file and save it to:
```
/home/claude/writing-style/style-profile.md
```

The style profile should be written as a set of **direct instructions Claude should follow** when writing on the user's behalf — not a third-person description of the user. Format it like this:

```markdown
# [User's name] — Writing Style Profile
*Generated from [N] sent emails on [date]*

## Core voice
[2–3 sentence summary of the overall feel and approach]

## Greeting rules
- Cold/formal outreach: ...
- Professional working contact: ...
- Close contact: ...
- Courtesy opener (if used): ...

## Tone by relationship
[Describe 2–4 tiers with specific examples from the emails]

## Request structure
[How they frame asks — e.g. context first, then specific need, then CTA]

## Structural habits
[Sentences, paragraphs, lists, email length by context]

## Sign-off rules
- Formal: ...
- Professional: ...
- Close: ...

## Vocabulary and phrasing
- Recurring phrases to use: ...
- Phrases to avoid entirely: ...

## Email length by context
- Cold outreach: ...
- Professional update/request: ...
- Quick reply: ...
```

Tell the user the profile has been saved and briefly summarise the key style traits you found (3–5 bullet points). Then ask if anything looks wrong or needs adjusting.

---

## MODE 2: WRITE — Draft content in the user's style

**When to run this mode:** The user wants to write an email, message, or other content in their own voice, and setup has already been run.

### Step 1: Load the style profile

Read `/home/claude/writing-style/style-profile.md`. If the file doesn't exist, switch to MODE 1 first, then come back.

### Step 2: Understand the writing task

Before drafting, confirm you know:
- Who is the recipient? (their relationship to the user — cold, professional, close)
- What is the purpose? (outreach, request, follow-up, reply, thank-you, update)
- What specific content needs to be included?
- Is there a reply to respond to, or is this an original email?

If the user hasn't given you enough to determine recipient relationship and purpose, ask one focused question before drafting.

### Step 3: Draft

Apply the style profile strictly. The goal is that the user should be able to send the draft with no more than minor edits. Specifically:

- Use the correct greeting for the relationship tier
- Apply the correct tone register throughout
- Use the user's sentence structure habits — don't over-polish or academicise
- Mirror their vocabulary (recurring phrases) and avoid what they avoid
- Use the correct sign-off
- Calibrate email length to what they'd write in this context
- Do NOT add your own flair, formal corporate phrases, or filler content they wouldn't use

### Step 4: Deliver

Present the draft cleanly. Don't add a long explanation of what you did — just present it and offer to adjust if anything doesn't sound right.

---

## Updating the style profile

If the user says "update my style" or "refresh my style from my latest emails", re-run MODE 1. This overwrites the existing style-profile.md with a fresh analysis.

---

## Notes for Claude

- The style profile belongs to the user — treat it as their voice, not a template
- When in doubt about register, err toward the more formal tier; the user can always loosen it
- If the user explicitly overrides a style rule for a specific email ("make this more casual than usual"), follow the override for that email only
- This skill requires Gmail to be connected for SETUP mode. If Gmail is not available, ask the user to paste a sample of 5–10 of their sent emails directly into chat and proceed with the analysis from that
