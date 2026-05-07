# HLD — Main Agent (Orchestrator)

## Overview

The main agent (`launchdarkly-agent`) is the orchestrator. It analyzes GitHub context (PR diffs, issue bodies, comments), detects feature flag opportunities, delegates flag creation to the sub-agent, and reports results back to the developer.

## Architecture

```
GitHub Event (PR, issue, @-mention)
        │
        ▼
  ┌─────────────────────┐
  │   Main Agent         │
  │   (launchdarkly-agent)│
  │                     │
  │  1. Analyze context  │
  │  2. Detect flag      │
  │     opportunities    │
  │  3. Check duplicates │──→ list-flags skill ──→ LaunchDarkly MCP
  │  4. Delegate creation│──→ flag-creator agent ──→ create-flag skill ──→ LaunchDarkly MCP
  │  5. Report back      │──→ PR/issue comment
  └─────────────────────┘
```

## Responsibilities

| Responsibility | Description |
|---|---|
| **Analyze context** | Read the PR diff, issue body, or comment that triggered the agent |
| **Detect opportunities** | Identify code patterns that benefit from feature flags |
| **Check duplicates** | Use `list-flags` skill to verify flag doesn't already exist |
| **Delegate creation** | Pass structured flag parameters to the `flag-creator` sub-agent |
| **Report back** | Post a GitHub comment with flag details, links, and next steps |
| **Escalate** | Ask clarifying questions if project key or flag type can't be determined |

## Trigger Sources

- `@`-mention in a PR, issue, or discussion
- Assignment to an issue or task
- Push or PR event (via platform triggers)

## Design Decisions

1. **Orchestrator pattern** — Main agent handles analysis and coordination; flag creation is delegated to a specialized sub-agent.
2. **Duplicate prevention** — Always list existing flags before creating new ones.
3. **Ask, don't guess** — If context is ambiguous, ask the user rather than creating incorrect flags.
