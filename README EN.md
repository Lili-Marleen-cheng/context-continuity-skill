# Context Continuity Skill

> Turn long AI conversations into a portable **Work State** that another chat or agent can continue safely.

**Core idea:** this is not conversation summarization. It is **work-state extraction**.

A summary explains what happened. A Work State tells the next agent what is true, what has been decided, what remains unresolved, and what to do next.

## Core flow

```text
Long conversation
      ↓
Extract Work State
      ↓
Human review
      ↓
Save / checkpoint
      ↓
Resume in a new chat or another AI
```

## Work State v1

The current Skill intentionally keeps the state small:

1. Goal
2. Current status
3. Confirmed decisions
4. Rejected approaches
5. Open questions
6. Next action
7. Constraints

The design principle is:

> Preserve what is necessary to continue the work, not everything that was said.

## Repository structure

```text
context-continuity-skill/
├── README.md
├── LICENSE
├── CHANGELOG.md
├── skills/
│   └── context-continuity/
│       ├── SKILL.md
│       └── templates/
│           ├── work-state.md
│           └── resume-prompt.md
├── examples/
│   └── example-work-state.md
└── docs/
    └── design.md
```

## Cross-agent

The Resume Prompt is plain text so it can be used across Claude, ChatGPT, Gemini, and other chat-based assistants.

## Philosophy

- **Reviewable** — the user sees the Work State before it is saved or handed off.
- **Portable** — the state is a plain Markdown artifact.
- **Low-friction** — the user should be able to continue work without managing a complicated memory system.

## Status

Early open-source Skill / protocol experiment.
