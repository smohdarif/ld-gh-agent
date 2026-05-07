# Learning — Flag Creator Sub-Agent

## Key Concepts

### Sub-Agent Pattern

The flag-creator is not standalone — it's always invoked by the main agent. It has its own MCP connection but no file system tools (`tools: []`), keeping it focused purely on LaunchDarkly API interactions.

### Create-then-Verify

The agent always calls `get_feature_flag` after `create_feature_flag`. This confirms the flag was actually persisted and allows the agent to extract the list of environments.

### Structured I/O

Unlike the main agent (which deals with natural language context), the flag-creator expects structured input and returns structured output. This makes the contract between agents clear and predictable.

### Error Transparency

The agent never retries silently. If something fails, it returns a clear error so the main agent can decide how to handle it (e.g., ask the user for a different project key).

### What Gets Created

- A **flag definition** in the specified project
- The flag is created in an **off** state across all environments
- Default `true`/`false` variations for boolean flags
- **No** targeting rules, rollouts, segments, or prerequisites
