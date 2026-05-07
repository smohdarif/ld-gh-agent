# PRD — Flag Creator Sub-Agent

## Problem Statement

Flag creation involves multiple API calls (create + verify) and structured error handling. Embedding this directly in the main agent would make the orchestrator too complex and harder to maintain.

## Goals

1. Provide a reliable, structured flag creation workflow
2. Verify flag creation was successful
3. Return clear, structured results (success/failure, URL, environments)
4. Handle errors gracefully without silent retries

## Requirements

| ID | Requirement | Priority |
|---|---|---|
| R1 | Accept structured input (project_key, flag_key, flag_name, etc.) | Must |
| R2 | Create flag via LaunchDarkly MCP `create_feature_flag` tool | Must |
| R3 | Verify creation via `get_feature_flag` | Must |
| R4 | Return structured result with success/failure, URL, environments | Must |
| R5 | Handle "flag already exists" gracefully | Must |
| R6 | Handle invalid project key with clear error | Must |
| R7 | Never retry silently — surface all errors | Must |
