# Learning — Skills Design Rationale

## Key Concepts

### Skills Are a GitHub Copilot Extensions Concept

Skills are **not** a Claude Code concept. They are part of the GitHub Copilot Extensions plugin specification. The `SKILL.md` format, the `skills/` directory convention, and how skills are discovered and invoked — all defined by the Copilot Extensions platform.

| Platform | Concept |
|---|---|
| GitHub Copilot Extensions | Skills (`SKILL.md` in `skills/` directory) |
| Claude Code | Skills (`/skill` command, different system entirely) |
| Both | Use the word "skill" but they are unrelated systems |

### Skills Don't Have Their Own Runtime

A skill is just a set of instructions. It doesn't:
- Start a new LLM instance
- Have its own MCP connection
- Have its own tools
- Run in a separate context

When an agent invokes a skill, the skill's instructions are loaded into the agent's context, and the agent's LLM follows them using the agent's own tools. Think of it as "importing instructions."

### Sub-Agents DO Have Their Own Runtime

When an agent invokes a sub-agent, the platform:
- Starts a new LLM instance
- Gives it the sub-agent's system prompt
- Provides its own MCP connections
- Returns a structured result when done

This is why the `flag-creator` sub-agent has `tools: []` and its own `mcp-servers` declaration — it runs independently.

### The Redundancy Is Intentional

The overlap between `flag-creator.agent.md` and `create-flag/SKILL.md` exists because they serve different invocation patterns:

- **Skill path**: Main agent → loads create-flag skill instructions → follows them in its own context → fast, no overhead
- **Sub-agent path**: Main agent → invokes flag-creator → separate LLM reasons about creation → structured error handling → result returned

The skill provides the *reusable instructions*. The sub-agent provides the *isolated execution environment*.

### When the Design Pays Off

The skills design becomes valuable when:

1. **A new agent is added** — e.g., a `flag-audit-agent` that reuses `list-flags` to find stale flags
2. **Skills are composed differently** — e.g., a cleanup workflow that lists flags, checks age, and deletes old ones
3. **The plugin grows** — more agents, more workflows, same core operations

With just one main agent and one sub-agent (the current state), the benefit is mostly architectural cleanliness and future-proofing.

## Common Misconception

"Skills are like functions" — Not exactly. Skills are more like **instruction templates**. When invoked, they don't execute in a sandbox; they inject instructions into the invoking agent's LLM context. The agent still decides how to follow them.
