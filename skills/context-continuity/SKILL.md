---
name: context-continuity
description: Use this skill whenever a conversation is getting long, the context window is close to running out, the user wants to continue the work in a new chat, hand work off to a different AI assistant, or is worried about losing track of decisions made earlier. Also trigger when the user asks to "save my progress", "generate a resume prompt", "checkpoint this", "continue where we left off", "回到正轨", "续接对话", "生成续接提示词", or when Claude notices signs of context drift — repeating a question the user already answered, re-proposing an approach the user already rejected, or contradicting a decision the user already confirmed. This skill extracts a portable, human-reviewable "Work State" snapshot (goal, confirmed decisions, rejected approaches, open questions, next actions) and produces a copy-paste-ready resume prompt that works in a new chat with Claude, or in ChatGPT, Gemini, or any other AI assistant.
---

# Context Continuity

## What this skill does

Long agent conversations degrade in two ways: the context window fills up, or the
model quietly drifts — forgetting a decision, re-litigating something already
settled, or losing track of what "done" looks like. This skill gives the user a
way to snapshot the *working state* of a conversation — not a transcript, not a
summary of what was said, but a compact description of what is true and what to
do next — so that a new chat, or a different AI product entirely, can pick the
work back up without re-reading everything or repeating mistakes.

The core distinction: a **summary** describes what happened. A **Work State**
tells the next agent what to do. Optimize every field for "can a new agent act
on this correctly," not "does this capture everything that was said."

## When to trigger

- The user says the conversation is getting long, or asks to continue in a new chat
- The user asks to save/checkpoint progress, or generate a resume/handoff prompt
- The user wants to move the task to a different AI tool
- The user says the assistant seems to have lost the thread, is repeating itself,
  or is re-suggesting something already rejected — offer to "restore" from the
  last snapshot rather than trying to fix it in-place
- Proactively, if you (Claude) notice you're about to ask something the user
  clearly already answered earlier in this same conversation — that's a signal
  worth surfacing to the user, with an offer to run this skill

Do not trigger this for short conversations or single-turn tasks — it only
earns its cost once there's an actual body of decisions and progress to lose.

## Core principle: extract state, don't compress text

Do not summarize the conversation chronologically ("first we discussed X, then
Y, then Z"). Instead, sort everything that was said into a small number of
fixed categories. Anything that doesn't clearly belong in one of these
categories should usually just be dropped — resist the urge to preserve
everything "just in case." A bloated WorkState is as useless as no WorkState.

## The WorkState format

Use this structure. Keep every field terse — bullet points, not paragraphs.
Omit empty fields rather than writing "none."

```markdown
# Work State: <short title>
Updated: <date>

## Goal
<One or two sentences: what is this work trying to accomplish, and what does
"done" look like?>

## Current status
<One or two sentences: where things stand right now>

## Confirmed decisions
- <Decision the user explicitly made or approved — state it as fact, not as a
  suggestion. Only put things here the user actually confirmed, not things
  Claude proposed that went unanswered.>

## Rejected approaches
- <Something that was considered and explicitly turned down, and why —
  so the next agent doesn't re-propose it>

## Open questions
- <Things that are still genuinely unresolved and need a decision>

## Next action
<The single next concrete step. Not a list of everything left to do — just
what should happen immediately when work resumes.>

## Constraints
- <Hard limits that must not be violated: budget, tech stack, deadlines,
  things the user explicitly ruled out>
```

That's it — seven fields. Resist adding more. In particular, do **not** add:
a confidence score (there's no reliable way to compute one), a version-history
tree (a single "Updated" timestamp with overwrite-on-update is enough for
almost every use case), or verbatim quote/citation tracking (high effort, and
the "confirmed decisions" field already captures what matters — *that*
something was decided, not the exact sentence that decided it).

## Step-by-step: generating a snapshot

1. Re-read the conversation and sort content into the seven fields above.
   When something is ambiguous, prefer putting it in "Open questions" over
   guessing which bucket it belongs in.
2. **Priority rule for conflicts**: if the user explicitly confirmed something
   and an AI (in this conversation or a prior one) later suggested something
   that contradicts it, the user's confirmation wins. Only overwrite a
   confirmed decision if the user *explicitly* changed their mind — an AI
   proposing an alternative is not enough, even if the user didn't object to it.
3. Draft the WorkState in a message using the template above, and show it to
   the user before saving anything. This is the single most important step —
   the whole point of this skill is that the user can review and correct the
   state before it gets used to steer future work. Never save or hand off a
   WorkState the user hasn't seen.
4. Once the user confirms (or edits) it, save it as a markdown file. Ask the
   user where they'd like it saved if it's not obvious from context — don't
   assume a location, and never write outside a location the user specified or
   the current working directory.
5. Also output a **Resume Prompt** (see below) directly in the chat, so the
   user can copy-paste it into a new conversation immediately without needing
   to locate the saved file.

## The Resume Prompt

This is what actually gets pasted into the next chat — with Claude or any
other AI. It's the WorkState, wrapped in explicit instructions for how a new
agent should treat it:

```
You are continuing existing work. Do not restart from scratch or ask the
user to re-explain the background — everything you need is below.

GOAL
<goal>

CURRENT STATUS
<current status>

CONFIRMED DECISIONS (treat as settled fact, do not revisit)
<decisions>

REJECTED APPROACHES (do not re-suggest these unless the user brings new
information)
<rejected>

OPEN QUESTIONS
<open questions>

NEXT ACTION
<next action>

CONSTRAINTS
<constraints>

If anything above seems insufficient or contradictory, ask the user a
targeted clarifying question rather than guessing — but otherwise, get
straight to work on the next action instead of re-summarizing the above
back to the user.
```

This block is deliberately plain text with no special formatting
requirements, so it works identically whether it's pasted into Claude,
ChatGPT, Gemini, or any other chat-based assistant. That plainness — not any
platform integration — is what makes it cross-agent: nothing about it depends
on Claude-specific features.

## Restoring from drift

If the user says the assistant seems to have lost the thread mid-conversation
(not starting a new chat, just righting the current one):

1. Ask the user to paste in (or point to) the most recent saved WorkState
   file, if one exists. If none exists, offer to generate one now from
   whatever's usable in the current conversation before doing anything else.
2. Re-state the Goal, Confirmed decisions, and Next action back to the user in
   a short message so they can confirm the assistant is now aligned, before
   continuing the actual task.
3. Do not silently continue as if nothing happened — the check-in matters more
   than the snapshot itself here, since the goal is to rebuild the user's
   trust that things are back on track.

## What this skill deliberately does not do

- It does not automatically detect a full context window or silently inject
  anything — every step here is user-visible and user-confirmed by design.
  Silent automation is a bigger risk (an unreviewed wrong "fact" propagating
  forward) than the small amount of friction this adds.
- It does not attempt real-time sync between AI platforms. The Resume Prompt
  is a portable artifact the user carries themselves; this skill does not
  read or write to any other AI product.
- It does not try to preserve everything said in the conversation. If it's not
  a goal, a confirmed decision, a rejection, an open question, a next action,
  or a constraint, it does not belong in the WorkState.
