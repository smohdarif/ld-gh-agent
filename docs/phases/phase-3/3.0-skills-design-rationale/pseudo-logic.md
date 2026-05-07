# Pseudo Logic — Skills Design Rationale

## How Skills Are Loaded and Executed

```
// ═══════════════════════════════════════════
// SKILL DISCOVERY (at plugin load time)
// ═══════════════════════════════════════════

WHEN platform loads plugin:
  skills_dir = plugin.json.skills  // "skills/"

  FOR each subdirectory in skills_dir:
    skill_file = {subdirectory}/SKILL.md
    IF skill_file exists:
      front_matter = parse_yaml(skill_file)
      body = parse_markdown(skill_file)
      REGISTER skill:
        name = front_matter.name
        description = front_matter.description
        instructions = body

  // Result: 3 skills registered
  //   - create-flag
  //   - list-flags
  //   - suggest-flags


// ═══════════════════════════════════════════
// SKILL INVOCATION (at runtime)
// ═══════════════════════════════════════════

WHEN agent LLM decides to use a skill:

  // Example: main agent wants to check for duplicates

  1. Agent LLM outputs: "I'll use the list-flags skill"

  2. Platform loads skill instructions:
     skill = get_skill("list-flags")
     instructions = skill.instructions

  3. Platform provides instructions to agent LLM as context:
     agent_context += instructions

  4. Agent LLM follows skill's steps using its OWN tools:
     // The skill says "Call list_feature_flags MCP tool"
     // The agent has MCP tools available
     // So the agent calls the MCP tool as instructed by the skill

  5. Agent LLM formats output per skill's output format:
     "Existing flags in project 'default':
      - `new-checkout-flow` — New Checkout Flow (created 2026-01-15)"


// ═══════════════════════════════════════════
// COMPARISON: SKILL vs SUB-AGENT INVOCATION
// ═══════════════════════════════════════════

// SKILL invocation (runs in agent's context):
WHEN agent uses create-flag SKILL:
  skill_instructions loaded into agent's LLM context
  agent's LLM follows instructions
  agent's MCP tools are used
  output stays in agent's context
  // Single LLM, single context

// SUB-AGENT invocation (runs in separate context):
WHEN agent calls flag-creator SUB-AGENT:
  platform starts new agent instance
  new LLM context with flag-creator's system prompt
  new MCP connection (independently authenticated)
  sub-agent executes autonomously
  result returned to calling agent
  // Two LLMs, two contexts
```

## When to Use a Skill vs a Sub-Agent

```
DECISION: skill or sub-agent?

IF task is simple, single-step, or read-only:
  → USE SKILL (runs in agent's context, faster, no overhead)
  → Examples: list-flags, suggest-flags

IF task is multi-step, needs error handling, or modifies state:
  → USE SUB-AGENT (isolated context, structured error contract)
  → Examples: flag-creator (create + verify + error handling)

IF task might be reused by future agents:
  → EXTRACT AS SKILL (even if also wrapped by a sub-agent)
  → Examples: create-flag skill (reusable instructions)
```
