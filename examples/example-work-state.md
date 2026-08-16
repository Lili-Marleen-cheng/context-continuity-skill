# Work State: Context Continuity MVP
Updated: 2026-08-16

## Goal
Build a lightweight agent work-state layer that lets a user continue unfinished AI work without manually managing context.

## Current status
The product direction is defined. The next task is implementing the first public Beta MVP.

## Confirmed decisions
- The product is not a generic conversation summarizer.
- The core abstraction is Work State.
- The primary user action is "New chat, continue".
- A recovery action should let the user return the agent to a stable state.
- Cross-agent handoff is a future capability.

## Rejected approaches
- Building only a HANDOFF.md generator — insufficient differentiation.
- Making the user manually manage memory — too much friction.

## Open questions
- Exact MVP architecture and deployment boundary
- How much evidence should be retained in the first release

## Next action
Implement the first public Beta MVP around Continue + Work State + Restore.

## Constraints
- Keep the user experience lightweight.
- Do not expose complex context-management mechanics to the user.
- Optimize for low infrastructure and LLM cost.
