# PRD — Skills Design Rationale

## Problem Statement

The plugin needs to perform three distinct operations: create flags, list flags, and suggest flags. These operations could be embedded directly in agents or extracted as reusable skills. The design choice impacts maintainability, reusability, and complexity.

## Goals

1. Define reusable operations that any agent in the plugin can invoke
2. Keep agents focused on decision-making, not API mechanics
3. Enable future extensibility (new agents reusing existing skills)
4. Minimize unnecessary duplication

## Requirements

| ID | Requirement | Priority |
|---|---|---|
| R1 | Skills must be discoverable from the `skills/` directory | Must |
| R2 | Skills must define clear parameters and output format | Must |
| R3 | Skills must be invocable by any agent in the plugin | Must |
| R4 | Skills should not duplicate agent logic unnecessarily | Should |
| R5 | The design should support adding new agents without duplicating skill logic | Should |

## Trade-offs

| Approach | Reusability | Simplicity | Duplication |
|---|---|---|---|
| Agents only | Low | High | None |
| Skills only | High | High | None |
| Both (current) | High | Medium | Some |

## Decision

The current design (Option C: both agents and skills) was chosen for extensibility, with the understanding that the `flag-creator` agent and `create-flag` skill have overlapping instructions. This is acceptable because:

- `list-flags` and `suggest-flags` are genuinely separate from any agent
- The `flag-creator` adds structured error handling beyond what the skill defines
- Future agents (e.g., cleanup, audit) can reuse skills without modification
