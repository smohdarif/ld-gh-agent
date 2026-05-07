# PRD — Suggest Flags Skill

## Problem Statement

Not every code change needs a feature flag. The agent needs a way to analyze context and identify only the changes that genuinely benefit from flag-based gating, before creating anything.

## Requirements

| ID | Requirement | Priority |
|---|---|---|
| R1 | Accept PR diff, issue body, or plain text as input | Must |
| R2 | Detect conditional logic, TODO comments, and new features | Must |
| R3 | Return structured recommendations with key, name, type, and rationale | Must |
| R4 | Prefer fewer, well-scoped flags over many granular ones | Must |
| R5 | Correctly classify temporary vs permanent flags | Should |
| R6 | Suggest which code to wrap with each flag | Should |
| R7 | Return "no opportunities" when none are found | Must |
