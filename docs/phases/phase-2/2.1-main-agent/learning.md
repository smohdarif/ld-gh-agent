# Learning — Main Agent (Orchestrator)

## Key Concepts

### Orchestrator Pattern

The main agent doesn't create flags directly. It analyzes context, makes decisions, and delegates the actual API calls to the `flag-creator` sub-agent. This separation keeps each component focused.

### LLM-Driven Analysis

The flag detection is not rule-based code — it's the LLM reading the PR diff and issue text, understanding the intent, and deciding if a feature flag would be useful. The agent's markdown instructions guide what patterns to look for.

### Context Sources

| Source | Available Data |
|---|---|
| PR diff | Added/removed/changed lines, file names |
| PR metadata | Title, description, branch name, author, labels |
| Issue | Title, body, labels, assignees |
| Comment | The text that @-mentioned the agent |

### Skills vs Sub-Agents

- **Skills** (`list-flags`, `suggest-flags`) are invoked directly by the main agent as focused tasks.
- **Sub-agents** (`flag-creator`) are separate agent instances with their own MCP connections and system prompts.

### Why Ask Instead of Guess

The agent is instructed to ask clarifying questions when it can't determine the project key or flag type. This is a deliberate design choice — creating a flag in the wrong project is worse than asking the developer which project to use.

## Common Patterns

### Good Flag Candidates
- New UI component behind a toggle
- New API endpoint that should be gradually rolled out
- Database migration that needs a kill switch
- A/B test implementation

### Not Flag Candidates
- Bug fixes
- Refactoring with no behavior change
- Documentation updates
- Test additions
