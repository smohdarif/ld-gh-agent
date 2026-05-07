# HLD — List Flags Skill

## Overview

The `list-flags` skill retrieves existing feature flags from a LaunchDarkly project. It's primarily used as a duplicate check before creating new flags.

## Flow

```
Agent invokes list-flags skill
        │
        ▼
  Call list_feature_flags (MCP)
  with project_key, search?, tag?, limit?
        │
        ▼
  Return formatted list:
    "Existing flags in project '{project_key}':
     - `flag-key` — Flag Name (created date)
     - ..."
```

## Parameters

| Parameter | Required | Default | Description |
|---|---|---|---|
| `project_key` | Yes | — | LaunchDarkly project to query |
| `search` | No | — | Text filter for flag name or key |
| `tag` | No | — | Tag filter (e.g., `repo:my-org/my-repo`) |
| `limit` | No | 20 | Maximum number of flags to return |

## Design Decisions

1. **Always called before create** — The main agent uses this to prevent duplicates.
2. **Highlights matches** — If a flag matching the intended key exists, it's called out clearly.
3. **Supports filtering** — Search by text or tag to narrow results.
