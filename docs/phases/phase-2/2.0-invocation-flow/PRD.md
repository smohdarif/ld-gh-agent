# PRD — Invocation Flow

## Problem Statement

The invocation chain spans multiple components (platform, agents, skills, MCP servers) with no traditional code connecting them. Understanding how these components communicate and what triggers what is critical for debugging, extending, and maintaining the plugin.

## Goals

1. Clear documentation of the full invocation chain from developer action to flag creation
2. Explain how the platform mediates agent-to-agent communication
3. Clarify what context is available at each stage
4. Document how the agent accesses the codebase without direct GitHub API calls

## Requirements

| ID | Requirement | Priority |
|---|---|---|
| R1 | Platform must detect @-mention, assignment, and PR events | Must |
| R2 | Pre-run hook must execute before any agent starts | Must |
| R3 | OIDC token exchange must complete before MCP tools are available | Must |
| R4 | Platform must inject PR/issue context into the agent | Must |
| R5 | Platform must make sub-agents available as callable tools | Must |
| R6 | Main agent must be able to invoke flag-creator and receive structured results | Must |
| R7 | Platform must post agent's response as a GitHub comment | Must |
| R8 | Platform must revoke OIDC token after job completion | Must |

## Key Constraints

- No direct agent-to-agent communication — all routing goes through the platform
- No direct GitHub API calls — agents use platform-provided tools (`view`, `edit`)
- No stored secrets — authentication is handled via OIDC token exchange per invocation
