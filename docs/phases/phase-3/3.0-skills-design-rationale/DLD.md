# DLD — Skills Design Rationale

## Detailed Design

### Skill File Format (Copilot Extensions Spec)

```
skills/
└── {skill-name}/
    └── SKILL.md
```

Each `SKILL.md` has:

```yaml
---
name: {skill-name}
description: {what the skill does}
disable-model-invocation: false
---

# {Skill Title}

{Natural language instructions: parameters, steps, output format}
```

### Skills in This Plugin

| Skill | Invoked By | MCP Tools Used | Purpose |
|---|---|---|---|
| `create-flag` | Main agent (or flag-creator) | `create_feature_flag`, `get_feature_flag` | Create + verify a flag |
| `list-flags` | Main agent | `list_feature_flags` | Check for duplicates |
| `suggest-flags` | Main agent | None (LLM analysis only) | Analyze context for flag opportunities |

### How Skills Are Invoked

Skills don't have their own runtime. When an agent invokes a skill:

1. The platform loads the skill's `SKILL.md`
2. The skill's instructions are provided to the agent's LLM as additional context
3. The LLM follows the skill's instructions using the agent's available tools
4. The output follows the skill's defined format

```
Main Agent LLM
    │
    │  "I need to check for duplicates"
    │  → loads list-flags/SKILL.md instructions
    │  → follows the skill's steps using its own MCP tools
    │  → produces output in the skill's defined format
    │
    │  "I need to create a flag"
    │  → loads create-flag/SKILL.md instructions
    │  → follows the skill's steps using its own MCP tools
    │  → produces output in the skill's defined format
```

### Overlap Analysis: flag-creator Agent vs create-flag Skill

#### flag-creator.agent.md
```markdown
## What you do
1. Call the LaunchDarkly MCP `create_feature_flag` tool
2. Verify the flag was created by calling `get_feature_flag`
3. Return a structured result
```

#### skills/create-flag/SKILL.md
```markdown
## Steps
1. Use the LaunchDarkly MCP tool `create_feature_flag`
2. Confirm creation by calling `get_feature_flag`
3. Return the flag key, name, and a direct URL
```

**The core logic is identical.** The differences:

| Aspect | flag-creator Agent | create-flag Skill |
|---|---|---|
| Has own MCP connection | Yes | No |
| Has structured error contract | Yes (success/error JSON) | No (text output) |
| Can be invoked independently | Yes (as a sub-agent) | No (needs an invoking agent) |
| Runs in own context | Yes (separate LLM call) | No (runs in invoking agent's context) |

### Alternative Designs

#### Option A: Agents Only (No Skills)

```
agents/
├── main.agent.md         ← embeds list/suggest logic directly
└── flag-creator.agent.md ← embeds create logic directly
skills/                   ← empty or removed
```

Pros: No duplication, fewer files
Cons: Logic not reusable, main agent becomes bloated

#### Option B: Skills Only (No Sub-Agent)

```
agents/
└── main.agent.md         ← calls skills directly
skills/
├── create-flag/SKILL.md
├── list-flags/SKILL.md
└── suggest-flags/SKILL.md
```

Pros: Simpler, less duplication
Cons: No isolated error handling for creation, main agent context gets large

#### Option C: Both (Current Design)

```
agents/
├── main.agent.md         ← calls skills + sub-agent
└── flag-creator.agent.md ← dedicated creation workflow
skills/
├── create-flag/SKILL.md  ← reusable creation instructions
├── list-flags/SKILL.md   ← reusable listing instructions
└── suggest-flags/SKILL.md ← reusable analysis instructions
```

Pros: Most extensible, clean separation
Cons: Some duplication between flag-creator and create-flag
