# HLD — Flag Creator Sub-Agent

## Overview

The `flag-creator` is a specialized sub-agent that handles the structured workflow for creating a single LaunchDarkly feature flag. It is invoked by the main agent with specific parameters and returns a structured result.

## Architecture

```
Main Agent
    │
    │  Invokes with: project_key, flag_key, flag_name,
    │                description, tags, flag_type, temporary
    ▼
┌─────────────────────┐
│   Flag Creator Agent │
│                     │
│  1. Create flag     │──→ create_feature_flag (MCP tool)
│  2. Verify creation │──→ get_feature_flag (MCP tool)
│  3. Return result   │
└─────────────────────┘
    │
    │  Returns: { success, flag_key, flag_url, environments, error }
    ▼
Main Agent (posts summary to PR/issue)
```

## Design Decisions

1. **Separation from main agent** — Flag creation is a distinct, structured task. Isolating it in a sub-agent keeps the main agent focused on analysis and coordination.
2. **Verify after create** — Always call `get_feature_flag` after creation to confirm the flag exists.
3. **Structured error handling** — Return clear error objects so the main agent can report failures meaningfully.
4. **No retry** — Surface errors to the calling agent rather than retrying silently.
