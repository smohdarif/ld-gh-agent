# Pseudo Logic — Suggest Flags Skill

```
FUNCTION suggest_flags(input):
  // input = PR diff, issue body, or plain text

  1. SCAN input for flag-worthy patterns:

     opportunities = []

     IF input contains new conditional logic (if/switch guarding new behavior):
       ADD opportunity: { reason: "new conditional logic", lines: ... }

     IF input contains flag-related comments ("TODO: gate this", "feature flag", "experiment"):
       ADD opportunity: { reason: "explicit developer intent", lines: ... }

     IF input contains entirely new user-facing functions/components:
       ADD opportunity: { reason: "new user-facing feature", lines: ... }

     IF input contains database migrations or API changes:
       ADD opportunity: { reason: "coordinated rollout needed", lines: ... }

     IF input contains A/B test or percentage rollout indicators:
       ADD opportunity: { reason: "experiment/rollout setup", lines: ... }

  2. IF opportunities is empty:
       RETURN "No feature flag opportunities detected in the provided context."

  3. CONSOLIDATE opportunities:
     // Prefer fewer, well-scoped flags
     // Merge related opportunities into single flags
     // Remove trivial suggestions

  4. FOR each consolidated opportunity:
     DERIVE:
       flag_name = descriptive name from context
       flag_key  = normalize(flag_name) → lowercase, hyphen-separated
       flag_type = "boolean" (default) or infer from context
       temporary = true (if rollout/experiment) or false (if kill switch)
       rationale = one sentence explaining why this needs a flag
       gate      = describe which code/behavior to wrap

  5. FORMAT and RETURN:
     "Suggested flags:\n"
     FOR each flag:
       "{n}. **{flag_name}** (`{flag_key}`)\n"
       "   Type: {flag_type} | temporary: {temporary}\n"
       "   Why: {rationale}\n"
       "   Gate: {gate}\n"
```
