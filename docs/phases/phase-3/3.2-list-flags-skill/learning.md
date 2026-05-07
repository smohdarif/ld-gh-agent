# Learning — List Flags Skill

## Key Concepts

### Duplicate Prevention

The main agent always calls `list-flags` before `create-flag`. This is the primary mechanism for preventing duplicate flags. The skill searches by flag key to find exact or near matches.

### Filtering

- **Search**: Free-text filter against flag name and key. Useful for finding flags related to a feature.
- **Tag**: Exact tag match. Useful for finding all flags related to a specific repo or PR.

### Pagination

The `limit` parameter defaults to 20. For projects with many flags, the agent may need to filter to get relevant results.
