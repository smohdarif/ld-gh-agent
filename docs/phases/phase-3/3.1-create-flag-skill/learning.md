# Learning — Create Flag Skill

## Key Concepts

### Skills in GitHub Copilot Extensions

A skill is a reusable task definition written in markdown. It doesn't have its own MCP connection — it uses the tools available to the agent that invokes it. Skills are simpler than agents: they define a focused workflow with clear input/output.

### Flag Types

| Type | Variations | Use Case |
|---|---|---|
| `boolean` | `true` / `false` | Feature toggles, kill switches |
| `string` | Custom string values | Feature variants ("control", "variant-a", "variant-b") |
| `number` | Custom number values | Numeric configuration (timeout, limit) |
| `json` | Custom JSON objects | Complex configuration |

### Temporary vs Permanent Flags

- **Temporary (`true`)**: Short-lived flags for rollouts and experiments. Should be cleaned up after full rollout.
- **Permanent (`false`)**: Long-lived operational controls like kill switches or circuit breakers.
