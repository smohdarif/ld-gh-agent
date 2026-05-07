# HLD — Suggest Flags Skill

## Overview

The `suggest-flags` skill analyzes a PR diff, issue description, or feature description and produces a structured list of feature flag recommendations. It's the "thinking" step before creation — used to identify flag opportunities without committing to action.

## Flow

```
Agent provides context (diff, issue body, or description)
        │
        ▼
  LLM analyzes input for flag patterns:
    - New conditional logic
    - TODO/gate comments
    - New user-facing components
    - Gradual rollout indicators
    - A/B test setups
        │
        ▼
  Return structured recommendations:
    "Suggested flags:
     1. **Flag Name** (`flag-key`)
        Type: boolean | temporary: true
        Why: ...
        Gate: ..."
```

## Design Decisions

1. **Analysis only** — This skill does not create flags. It only recommends them.
2. **Fewer, well-scoped flags** — Prefer quality over quantity. Don't suggest a flag for every changed line.
3. **Temporary vs permanent** — Mark rollout/experiment flags as temporary, kill switches as permanent.
4. **Explicit "nothing found"** — If no opportunities exist, say so clearly.
