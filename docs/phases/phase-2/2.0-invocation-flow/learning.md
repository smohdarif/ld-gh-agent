# Learning — Invocation Flow

## Key Concepts

### Platform as Orchestrator

The GitHub Copilot Extensions platform is the central orchestrator. It:
- Detects GitHub events and matches them to plugins
- Runs lifecycle hooks
- Handles authentication (OIDC token exchange)
- Injects context into agents
- Routes calls between agents
- Posts results back to GitHub
- Cleans up (token revocation)

No agent communicates directly with another agent or with GitHub. Everything goes through the platform.

### LLM-Driven Decisions

The agents don't follow hardcoded control flow. The LLM reads the natural language instructions in the agent's markdown body and makes decisions:
- "Should I create a flag for this code change?" → LLM decides
- "What should the flag key be?" → LLM derives from context
- "Should I ask the user or proceed?" → LLM decides based on confidence

### Agents as Tools

The platform registers sub-agents as tools available to the main agent. From the main agent's perspective, calling the `flag-creator` sub-agent is no different from calling any other tool — it provides input and receives output.

### Context Flow

```
GitHub Event
    │
    ▼
Platform gathers: PR diff, title, branch, comments, etc.
    │
    ▼
Main agent receives context + reads more via "view" tool
    │
    ▼
Main agent passes structured params to flag-creator
    │
    ▼
Flag-creator uses params to call LaunchDarkly MCP tools
    │
    ▼
Result flows back: flag-creator → main agent → platform → GitHub comment
```

### Why Two Agents Instead of One?

The main agent handles **analysis** (understanding code, making decisions). The flag-creator handles **execution** (API calls, verification). Separating them:
- Keeps each agent focused
- Allows the flag-creator to be reused independently
- Isolates API errors from the analysis logic
- Makes the flag-creator testable in isolation

### No Direct GitHub API Access

The agents never call `octokit`, `gh`, or the GitHub REST API. They access code through:
1. **Platform-injected context** — PR diff, metadata (automatic)
2. **`view` tool** — Read files in the repo (on-demand)
3. **`edit` tool** — Modify files in the repo (on-demand)

This abstraction means the agents work across GitHub's infrastructure without managing API tokens or rate limits.
