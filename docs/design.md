# Design Notes

## Core distinction

Conversation Summary:
> describes what happened.

Work State:
> describes what is true for continuing the work.

## Why the current Skill stays small

The source Skill deliberately uses seven fields and avoids confidence scores, version trees, and verbatim citation tracking. This is intentional for the first release: prove that a compact, human-reviewable state is sufficient before adding heavier state infrastructure.

## Possible future layers

- Checkpoints
- Evidence / provenance
- Incremental state merge
- Conflict detection
- State alignment / drift detection
- Cross-agent adapters

These should be extensions, not requirements of the minimal Skill contract.
