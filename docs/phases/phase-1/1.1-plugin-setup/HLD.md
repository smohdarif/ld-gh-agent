# HLD — Plugin Setup & Configuration

## Overview

The plugin manifest (`plugin.json`) is the entry point for the GitHub Copilot Extensions platform. It declares the plugin's identity, version, and points to the agents, skills, and hooks that make up the system.

## Architecture

```
GitHub Copilot Extensions Platform
        │
        ▼
   plugin.json (entry point)
        │
        ├── agents/        → Agent definitions (markdown + YAML)
        ├── skills/        → Skill definitions (markdown)
        └── hooks.json     → Pre-run hook configuration
```

## Components

| Component | File | Purpose |
|---|---|---|
| Plugin Manifest | `plugin.json` | Declares plugin name, version, author, and paths to agents, skills, hooks |
| Hooks Config | `hooks.json` | Registers pre-run validation hooks |
| Gitignore | `.gitignore` | Excludes `.env`, `.DS_Store`, and PDFs from version control |

## Design Decisions

1. **Declarative architecture** — The entire plugin is configuration + natural language. No application code.
2. **Separation of concerns** — Agents, skills, and hooks are in separate directories, referenced by `plugin.json`.
3. **Single entry point** — `plugin.json` is the only file the platform needs to discover everything else.

## Dependencies

- GitHub Copilot Extensions platform (runtime)
- LaunchDarkly MCP server (external service)
