# PRD — List Flags Skill

## Problem Statement

Creating duplicate flags pollutes the LaunchDarkly project and causes confusion. The agent needs a way to check for existing flags before creating new ones.

## Requirements

| ID | Requirement | Priority |
|---|---|---|
| R1 | Query flags by project key | Must |
| R2 | Support text search filter | Should |
| R3 | Support tag filter | Should |
| R4 | Return flag key, name, and creation date | Must |
| R5 | Highlight flags that match the intended key | Must |
| R6 | Handle empty results gracefully | Must |
