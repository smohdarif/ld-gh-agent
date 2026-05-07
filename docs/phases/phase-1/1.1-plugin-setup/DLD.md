# DLD — Plugin Setup & Configuration

## Detailed Design

### plugin.json

```json
{
  "name": "launchdarkly-flag-automation",
  "description": "Automatically create and manage LaunchDarkly feature flags from GitHub pull requests and issues",
  "version": "1.0.0",
  "author": {
    "name": "LaunchDarkly",
    "email": "integrations@launchdarkly.com"
  },
  "license": "MIT",
  "agents": "agents/",
  "skills": "skills/",
  "hooks": "hooks.json"
}
```

### Field Details

| Field | Type | Description |
|---|---|---|
| `name` | string | Unique plugin identifier used by the platform |
| `description` | string | Human-readable description shown in the plugin marketplace |
| `version` | string | Semantic version of the plugin |
| `author.name` | string | Publisher name |
| `author.email` | string | Contact email |
| `license` | string | Open source license |
| `agents` | string | Relative path to the directory containing `.agent.md` files |
| `skills` | string | Relative path to the directory containing `SKILL.md` files |
| `hooks` | string | Relative path to the hooks configuration JSON file |

### hooks.json

```json
{
  "hooks": [
    {
      "event": "before:agent:run",
      "command": "scripts/validate-ld-context.sh"
    }
  ]
}
```

- **`event: before:agent:run`** — Fires before any agent in this plugin starts executing.
- **`command`** — Path to the shell script to execute. A non-zero exit code aborts the agent run.

### Directory Structure

```
ld-gh-agent/
├── plugin.json
├── hooks.json
├── .gitignore
├── agents/
│   ├── main.agent.md
│   └── flag-creator.agent.md
├── skills/
│   ├── create-flag/SKILL.md
│   ├── list-flags/SKILL.md
│   └── suggest-flags/SKILL.md
└── scripts/
    └── validate-ld-context.sh
```
