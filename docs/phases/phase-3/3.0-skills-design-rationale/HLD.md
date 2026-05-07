# HLD — Skills Design Rationale

## Overview

This document explains what skills are, why they exist as separate components, and honestly evaluates whether the separation is justified in this codebase.

## What Are Skills?

Skills are a **GitHub Copilot Extensions** concept (not Claude Code). They are reusable task definitions written in markdown with YAML front matter, discovered from the `skills/` directory declared in `plugin.json`.

```json
// plugin.json
{
  "agents": "agents/",
  "skills": "skills/",    ← Copilot Extensions plugin spec
  "hooks": "hooks.json"
}
```

Each skill has a `SKILL.md` file with:
- **Front matter** — `name`, `description`, `disable-model-invocation`
- **Body** — Natural language instructions defining parameters, steps, and output format

Skills don't have their own MCP connections or tools. They use whatever tools the invoking agent has access to.

## Skills vs Agents

| Aspect | Agent | Skill |
|---|---|---|
| **Has own MCP connection** | Yes | No (uses invoking agent's) |
| **Has own tools** | Yes (`tools: [...]`) | No |
| **Has own system prompt** | Yes | Yes (but simpler) |
| **Can invoke other agents** | Yes (via platform) | No |
| **Can be invoked by** | Platform, other agents | Agents only |
| **Purpose** | Autonomous reasoning + decision-making | Focused, reusable task |

## Architecture

```
plugin.json
    ├── agents/
    │   ├── main.agent.md ──────── can invoke ──→ skills + sub-agents
    │   └── flag-creator.agent.md  can invoke ──→ MCP tools directly
    │
    └── skills/
        ├── create-flag/SKILL.md ─── reusable by any agent
        ├── list-flags/SKILL.md ──── reusable by any agent
        └── suggest-flags/SKILL.md ── reusable by any agent
```

## Why Separate Skills?

### Theoretical Benefits

1. **Reusability** — Any agent in the plugin can invoke any skill. If you add a second agent (e.g., `flag-cleanup-agent`), it can reuse `list-flags` without duplicating instructions.

2. **Composability** — The main agent references skills by name. The LLM decides which skills to invoke based on context:
   ```
   "Use the list-flags skill first to check duplicates"
   "Use the create-flag skill to create the flag"
   ```

3. **Separation of concerns** — Agents decide *when and why*. Skills define *how*.

4. **Independent updates** — Change a skill's output format without touching any agent.

### When Skills Shine (Multiple Agents)

```
                        ┌─── main agent ──────── uses list-flags skill
                        │                   ├─── uses suggest-flags skill
                        │                   └─── uses create-flag skill
Plugin ────── skills ───┤
                        │                   ┌─── uses list-flags skill (reused!)
                        └─── cleanup agent ─┴─── uses delete-flag skill
                             (hypothetical)
```

### Honest Assessment for This Codebase

In the current codebase, there is **redundancy** between the `flag-creator` sub-agent and the `create-flag` skill:

| Component | Instructions |
|---|---|
| `flag-creator.agent.md` | "Call `create_feature_flag`, then verify with `get_feature_flag`" |
| `skills/create-flag/SKILL.md` | "Call `create_feature_flag`, then verify with `get_feature_flag`" |

With only one main agent and one sub-agent, the skills add modularity but also duplication. The design would be equally valid with:

- **Option A**: Keep only agents — embed skill logic in the flag-creator agent
- **Option B**: Keep only skills — remove the flag-creator agent, have the main agent call skills directly
- **Option C** (current): Both exist — the flag-creator is a thin wrapper that uses the same MCP tools

The current design is **forward-looking** — it pays off when more agents are added to the plugin.

## Design Decision

The project chose **Option C** because:

1. `list-flags` and `suggest-flags` are used directly by the main agent (not through the sub-agent), so they need to exist as skills regardless
2. `create-flag` as a skill keeps symmetry with the other skills
3. The `flag-creator` sub-agent adds structured error handling and verification on top of the raw skill
4. The pattern is extensible for future agents
