# Learning — Suggest Flags Skill

## Key Concepts

### Analysis vs Action

This skill only analyzes and recommends — it never creates flags. This gives the developer or main agent a chance to review before committing. The main agent can then selectively create flags from the suggestions.

### LLM-Driven Pattern Matching

The detection isn't regex-based. The LLM reads the diff/issue and uses its understanding of software engineering patterns to identify flag candidates. The skill's instructions guide what to look for, but the actual analysis is semantic.

### Quality over Quantity

The skill is instructed to prefer fewer, well-scoped flags. Creating too many flags adds operational overhead. A single well-placed flag is better than five granular ones.

### Temporary vs Permanent Decision Guide

| Scenario | temporary | Reason |
|---|---|---|
| New feature rollout | `true` | Will be removed after 100% rollout |
| A/B experiment | `true` | Will be removed after experiment concludes |
| Kill switch | `false` | Permanent safety mechanism |
| Operational control | `false` | Long-lived configuration toggle |
