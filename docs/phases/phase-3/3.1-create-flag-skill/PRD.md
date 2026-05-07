# PRD — Create Flag Skill

## Problem Statement

Creating a flag requires calling the LaunchDarkly API with the correct parameters and verifying the result. This logic should be reusable and well-defined.

## Requirements

| ID | Requirement | Priority |
|---|---|---|
| R1 | Accept project_key, flag_key, flag_name, and description | Must |
| R2 | Support optional tags, temporary, and flag_type | Should |
| R3 | Create flag via MCP `create_feature_flag` tool | Must |
| R4 | Verify creation via MCP `get_feature_flag` tool | Must |
| R5 | Return formatted output with flag details and URL | Must |
| R6 | Handle "already exists" case without error | Must |
