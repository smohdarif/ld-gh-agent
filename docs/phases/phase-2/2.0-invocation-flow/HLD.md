# HLD — Invocation Flow

## Overview

This document explains the complete invocation chain — from a developer action in GitHub to a feature flag being created in LaunchDarkly and a summary comment posted back on the PR/issue.

There is **no application code** connecting the components. The **GitHub Copilot Extensions platform** is the orchestrator — it detects events, invokes agents, routes calls between agents, and posts results.

## Architecture

```
┌─────────────┐     ┌──────────────────────┐     ┌──────────────────┐     ┌──────────────┐
│  Developer   │     │  GitHub Copilot      │     │  Main Agent      │     │  Flag Creator│
│  (GitHub UI) │     │  Extensions Platform │     │  (LLM)           │     │  Sub-Agent   │
└──────┬───────┘     └──────────┬───────────┘     └────────┬─────────┘     └──────┬───────┘
       │                        │                          │                      │
       │  @-mention / PR event  │                          │                      │
       │───────────────────────>│                          │                      │
       │                        │                          │                      │
       │                        │  Run pre-hook            │                      │
       │                        │  (validate-ld-context.sh)│                      │
       │                        │──┐                       │                      │
       │                        │<─┘ ✓                     │                      │
       │                        │                          │                      │
       │                        │  OIDC token exchange     │                      │
       │                        │  with LaunchDarkly       │                      │
       │                        │──────────────────────────│──────────────────────>│
       │                        │                          │                      │
       │                        │  Start main agent with   │                      │
       │                        │  context + tools + MCP   │                      │
       │                        │─────────────────────────>│                      │
       │                        │                          │                      │
       │                        │                          │  Analyze PR diff     │
       │                        │                          │  Detect flag opps    │
       │                        │                          │  Check duplicates    │
       │                        │                          │  (list-flags skill)  │
       │                        │                          │                      │
       │                        │                          │  Invoke flag-creator │
       │                        │                          │─────────────────────>│
       │                        │                          │                      │
       │                        │                          │                      │ create_feature_flag
       │                        │                          │                      │ (MCP tool)
       │                        │                          │                      │
       │                        │                          │                      │ get_feature_flag
       │                        │                          │                      │ (MCP tool → verify)
       │                        │                          │                      │
       │                        │                          │  { success, url }    │
       │                        │                          │<─────────────────────│
       │                        │                          │                      │
       │                        │  Post PR comment         │                      │
       │                        │<─────────────────────────│                      │
       │                        │                          │                      │
       │  Comment on PR:        │                          │                      │
       │  "Created flag..."     │                          │                      │
       │<───────────────────────│                          │                      │
       │                        │                          │                      │
       │                        │  Revoke OIDC token       │                      │
       │                        │──────────────────────────│──────────────────────>│
```

## Invocation Stages

### Stage 1: Developer Action

The developer performs an action in GitHub that triggers the agent:

| Action | Event Type |
|---|---|
| Types `@launchdarkly-agent` in a PR comment | @-mention |
| Assigns the agent to an issue | Assignment |
| Opens or pushes to a PR | Push/PR event (if configured) |

### Stage 2: Platform Bootstrapping

The GitHub Copilot Extensions platform:

1. **Detects the event** and matches it to the `launchdarkly-agent` plugin
2. **Runs the pre-hook** (`before:agent:run` → `validate-ld-context.sh`) to verify authentication is available
3. **Performs OIDC token exchange** with LaunchDarkly's MCP server to get an access token
4. **Injects context** — PR diff, title, description, branch, comments (or issue context)
5. **Starts the main agent** with the injected context, platform tools (`view`, `edit`), and authenticated MCP connection

### Stage 3: Main Agent Analysis

The main agent (LLM) autonomously:

1. Reads the PR diff / issue context
2. Identifies code changes that need feature flags
3. Derives flag parameters (key, name, tags, type)
4. Calls the `list-flags` skill to check for duplicates
5. If no duplicate exists, calls the `flag-creator` sub-agent

### Stage 4: Sub-Agent Flag Creation

The flag-creator sub-agent:

1. Receives structured parameters from the main agent
2. Calls `create_feature_flag` MCP tool → LaunchDarkly API
3. Calls `get_feature_flag` MCP tool → verifies creation
4. Returns structured result to the main agent

### Stage 5: Report Back

The main agent:

1. Receives the result from the flag-creator
2. Composes a summary comment
3. Posts the comment on the PR/issue via the platform

### Stage 6: Cleanup

The platform:

1. Revokes the OIDC access token
2. Marks the job as complete

## How Agent-to-Agent Communication Works

There is **no code, import, or function call** connecting the main agent to the sub-agent. The connection is entirely platform-mediated:

1. **Both agents are registered** in the same plugin (discovered from the `agents/` directory via `plugin.json`)
2. **The platform makes sub-agents available as tools** to the main agent
3. **The main agent's LLM decides** when to call the flag-creator, based on its natural language instructions
4. **The platform handles routing** — starts the sub-agent, passes parameters, returns the result
5. **Each agent has its own MCP connection** — both connect to the same LaunchDarkly MCP server but independently

The platform acts as a **message bus** — agents communicate through it, not directly.

## How the Agent Accesses the Codebase

The agent does **not** use the GitHub REST API or `git` commands directly:

| Channel | How | What |
|---|---|---|
| **Platform-injected context** | Automatic on trigger | PR diff, title, description, branch, changed files, comments |
| **`view` tool** | Declared in main agent front matter | Read any file in the repository |
| **`edit` tool** | Declared in main agent front matter | Modify files in the repository |
| **MCP tools** | Via LaunchDarkly MCP server | `create_feature_flag`, `list_feature_flags`, `get_feature_flag` |

Note: The flag-creator sub-agent has `tools: []` — no codebase access, only LaunchDarkly MCP tools.
