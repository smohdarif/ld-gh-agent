# PRD — Main Agent (Orchestrator)

## Problem Statement

Developers often forget to create feature flags when introducing new features, or create them inconsistently (wrong naming, missing tags, no description). The flag creation happens in a separate tool (LaunchDarkly dashboard) disconnected from where the code changes happen (GitHub).

## Goals

1. Automatically detect when a PR or issue would benefit from a feature flag
2. Create correctly-named, well-documented flags without leaving GitHub
3. Prevent duplicate flag creation
4. Give developers actionable next steps (which lines to wrap, SDK integration)

## Requirements

| ID | Requirement | Priority |
|---|---|---|
| R1 | Agent must analyze PR diffs to detect flag-worthy changes | Must |
| R2 | Agent must analyze issue descriptions for flag opportunities | Must |
| R3 | Agent must check for existing flags before creating new ones | Must |
| R4 | Agent must follow naming conventions (lowercase, hyphen-separated keys) | Must |
| R5 | Agent must tag flags with repo name and PR/issue number | Must |
| R6 | Agent must post a summary comment with flag details and next steps | Must |
| R7 | Agent must ask for clarification if project key is ambiguous | Must |
| R8 | Agent should suggest which code lines to wrap with the flag | Should |

## User Stories

1. As a developer, I @-mention the agent in a PR and it creates the right feature flag for me.
2. As a developer, I see a comment on my PR with the flag key, dashboard link, and which lines to gate.
3. As a team lead, I know all flags follow consistent naming and are tagged to their originating PR.
