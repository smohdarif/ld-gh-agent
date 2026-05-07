# Learning — Plugin Setup & Configuration

## Key Concepts

### GitHub Copilot Extensions Plugin

A plugin is a package of agents, skills, and hooks that extends GitHub Copilot's capabilities. The platform reads `plugin.json` to discover and register all components.

### Agents vs Skills

- **Agent** — An autonomous entity with its own system prompt, tools, and MCP server connections. It can reason, make decisions, and invoke skills.
- **Skill** — A reusable, focused task definition. Skills don't have their own MCP connections — they are invoked by agents and use the agent's tools.

### MCP (Model Context Protocol)

A protocol that allows AI agents to connect to external services and use their APIs as "tools". Instead of writing API client code, the agent declares an MCP server and gets tools injected automatically.

### Hooks

Hooks are shell commands that run in response to lifecycle events (e.g., `before:agent:run`). They can abort an agent run by exiting with a non-zero code.

### OIDC (OpenID Connect)

A federated authentication mechanism. GitHub signs a JWT with identity claims, sends it to a 3rd party token endpoint, and receives an access token in return. No long-lived secrets are stored.

## Useful References

- GitHub Copilot Extensions documentation
- MCP specification
- LaunchDarkly API documentation
- RFC 7523 (JWT bearer assertion)
- RFC 7009 (token revocation)
