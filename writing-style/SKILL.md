---
name: writing-style
description: >
  Capture and apply a user's personal writing style based on their sent emails
  and WhatsApp messages, then use it to write emails, WhatsApp messages, and
  other content that sounds exactly like them. ALWAYS use this skill when the
  user asks to "write in my style", "write this like I would", "write an email
  for me", "write a WhatsApp message for me", "sound like me", "capture my
  writing style", "set up my style", "update my style", or wants any written
  communication that matches their voice. Also trigger when the user wants to
  draft a professional email, reply, WhatsApp message, or other message and
  hasn't specified a different style — default to checking for a saved style
  profile first. Two modes: SETUP (reads sent emails and WhatsApp messages,
  creates and saves a style profile) and WRITE (loads the saved profile and
  applies it to draft content).
---

# Writing Style Skill

This skill has two modes. Read the user's intent and jump to the right one.

---

## MODE 1: SETUP — Capture the user's style

**When to run this mode:** The user says something like "capture my writing
style", "set up my style", "update my style from my emails", or they haven't
run setup before and want to write in their style.

### Step 1: Read sent emails via Gmail

Use the Gmail `search_threads` tool to retrieve the 30 most recent sent emails:
```
query: "in:sent"
pageSize: 30
view: THREAD_VIEW_MINIMAL
```

Then use `get_thread` with `messageFormat: FULL_CONTENT` on a sample of 8–12
of the most substantive threads (skip threads with only one-word/auto replies,
and threads where the user is clearly just forwarding something). Aim for
variety across: cold outreach, professional working relationship emails,
personal/close contact emails, and quick replies.

### Step 2: Read sent WhatsApp messages (if connected)

If the WhatsApp MCP is available, use `whatsapp: list_messages` to pull recent
messages sent by the user:

```
limit: 100
sort_by: newest
```

Filter to messages where `from_me = true`. Sample across different chat types:
- Individual chats with close contacts (friends, family)
- Individual chats with professional contacts
- Group chats

WhatsApp is particularly valuable for capturing the user's casual register —
how they write to people they're close to, which rarely shows up in email.
Look specifically at: greeting style (or lack of one), emoji usage, sentence
length, abbreviations, and how they open and close conversations.

If WhatsApp is not connected, skip this step and note in the profile that
the casual/close-contact register was inferred from email only.

### Step 3: Analyse the style

Read through the user's own messages from both sources. Extract the following:

**1. Greeting patterns**
- Email to unknown/formal: (e.g. "Good morning." / "Good day, [Name].")
- Email to working contacts: (e.g. "Hi [Name],")
- Email to close contacts: (e.g. "Hey [Name]!")
- WhatsApp opener: (e.g. "Hey", "Yo", no greeting at all, jumps straight in)
- Do they follow email greetings with a courtesy line?

**2. Tone register by relationship and channel**
- Map out 3–5 relationship/channel tiers: formal email → professional email →
  casual email → WhatsApp professional → WhatsApp close contact
- Describe the tone shift between each tier
- Note formal vs semi-formal vs casual markers per tier

**3. Sentence and paragraph structure**
- Email: average sentence length, front-loaded or build-up, request structure,
  use of lists vs prose
- WhatsApp: message length, single vs multi-message style, use of line breaks

**4. Sign-off patterns**
- Email closings (e.g. "Kind regards," / "Thanks," / "Thank you!")
- Name format in email (first name only, full name, with/without period)
- WhatsApp closings (e.g. nothing, "👍", "cheers", emoji)

**5. Distinctive vocabulary and phrasing**
- Recurring phrases or sentence starters unique to their voice (per channel)
- Words they avoid in email (corporate jargon, filler phrases)
- WhatsApp-specific patterns: abbreviations, emoji habits, filler words

**6. What they never do**
- Absent patterns in email: no hollow filler phrases, no sycophantic openers
- Absent patterns in WhatsApp: things that would feel out of character

**7. Formatting habits**
- Email: whitespace, paragraph breaks, numbered/unnumbered lists, length
- WhatsApp: single message vs multiple short messages, emoji placement

### Step 4: Write and save the style profile

Generate a style guide as a markdown file and save it to:
```
/home/claude/writing-style/style-profile.md
```

The style profile should be written as **direct instructions Claude should
follow** when writing on the user's behalf — not a third-person description.
Format it like this:

```markdown
# [User's name] — Writing Style Profile
*Generated from [N] sent emails and [N] WhatsApp messages on [date]*

## Core voice
[2–3 sentence summary of the overall feel and approach across all channels]

## EMAIL STYLE

### Greeting rules
- Cold/formal outreach: ...
- Professional working contact: ...
- Close contact: ...
- Courtesy opener (if used): ...

### Tone by relationship
[Describe 2–4 tiers with specific examples from the emails]

### Request structure
[How they frame asks — e.g. context first, then specific need, then CTA]

### Structural habits
[Sentences, paragraphs, lists, email length by context]

### Sign-off rules
- Formal: ...
- Professional: ...
- Close: ...

### Vocabulary and phrasing
- Recurring phrases to use: ...
- Phrases to avoid entirely: ...

### Email length by context
- Cold outreach: ...
- Professional update/request: ...
- Quick reply: ...

## WHATSAPP STYLE

### Opener
[How they start WhatsApp conversations — greeting or straight in, examples]

### Tone and register
[How casual, what markers distinguish their WhatsApp voice from email]

### Message structure
[Single long message vs multiple short ones, use of line breaks]

### Emoji usage
[Which emojis, how often, where in the message]

### Abbreviations and casual language
[Specific abbreviations or casual phrases they use — e.g. "lol", "lekker", "awe"]

### Closing
[How they end WhatsApp conversations — sign-off, emoji, or nothing]

### WhatsApp length by context
- Quick reply: ...
- Making plans: ...
- Group chat: ...
```

Tell the user the profile has been saved and briefly summarise the key style
traits you found (3–5 bullet points), noting any interesting differences
between their email and WhatsApp voice. Ask if anything looks wrong or needs
adjusting.

---

## MODE 2: WRITE — Draft content in the user's style

**When to run this mode:** The user wants to write an email, WhatsApp message,
or other content in their own voice, and setup has already been run.

### Step 1: Load the style profile

Read `/home/claude/writing-style/style-profile.md`. If the file doesn't exist,
switch to MODE 1 first, then come back.

### Step 2: Understand the writing task

Before drafting, confirm you know:
- **Channel:** Is this an email or a WhatsApp message? (This determines which
  section of the style profile to apply)
- **Recipient:** Who is this going to, and what's their relationship to the
  user? (cold, professional, close friend, family, group)
- **Purpose:** What does the message need to achieve?
- **Content:** What specific information needs to be included?
- **Context:** Is there a message to reply to, or is this an original?

If the channel isn't clear from context, ask. Everything else can be inferred
if needed.

### Step 3: Draft

Apply the correct section of the style profile for the channel:

**For email:** follow the EMAIL STYLE section strictly
**For WhatsApp:** follow the WHATSAPP STYLE section strictly

The goal is that the user should be able to send the draft with no more than
minor edits. Specifically:
- Use the correct greeting for the relationship tier and channel
- Apply the correct tone register throughout
- Mirror their sentence structure and length habits for that channel
- Use their vocabulary and avoid what they avoid
- Apply the correct sign-off or closing for the channel
- Calibrate length to what they'd write in this context
- Do NOT add your own flair, formal phrases, or filler they wouldn't use
- For WhatsApp: match emoji usage and casualness level exactly

### Step 4: Deliver

Present the draft cleanly. For WhatsApp messages, format it as it would
actually be sent (line breaks, emoji in place). Don't add a long explanation —
just present it and offer to adjust if anything doesn't sound right.

---

## Updating the style profile

If the user says "update my style" or "refresh my style", re-run MODE 1. This
overwrites the existing style-profile.md with a fresh analysis from the latest
emails and WhatsApp messages.

---

## Notes for Claude

- The style profile belongs to the user — treat it as their voice, not a
  template
- Email and WhatsApp are different channels — never bleed WhatsApp casualness
  into a formal email or vice versa
- When in doubt about register for email, err toward the more formal tier
- When in doubt about register for WhatsApp, err toward the more casual tier
- If the user explicitly overrides a style rule ("make this more formal than
  usual"), follow the override for that message only
- This skill requires Gmail for SETUP mode. If Gmail is not available, ask the
  user to paste sample messages directly into chat
- WhatsApp is optional — if not connected, the profile will still work well
  for email; the WhatsApp section will note it was not available
