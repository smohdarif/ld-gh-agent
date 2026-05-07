# Pseudo Logic — Main Agent (Orchestrator)

## Main Agent Flow

```
WHEN agent is triggered (via @-mention, PR event, issue assignment):

  1. DETERMINE trigger type:
     - PR comment/mention → read PR diff + comment body
     - Issue mention → read issue title + body
     - PR event → read PR diff + title + description

  2. ANALYZE context for flag opportunities:
     SCAN for:
       - New conditional logic (if/switch) guarding new behavior
       - Comments: "TODO: gate this", "feature flag", "experiment", "rollout"
       - Entirely new user-facing functions/components
       - Database migrations or API changes needing coordinated rollout
       - A/B test or percentage-based rollout descriptions

     IF no flag opportunities found:
       POST comment: "No feature flag opportunities detected."
       RETURN

  3. FOR each detected flag opportunity:

     a. DERIVE flag parameters:
        flag_key = normalize(branch_name OR feature_description)
          → lowercase, replace spaces/special chars with hyphens
          → strip issue numbers
        flag_name = PR_title OR issue_title
        description = one-sentence summary of what the flag gates
        tags = ["repo:{org}/{repo}", "pr:{number}" OR "issue:{number}"]
        flag_type = "boolean" (default, unless context suggests otherwise)
        temporary = true (for rollouts/experiments) OR false (for kill switches)

     b. CHECK for duplicates:
        INVOKE list-flags skill:
          project_key = {inferred or ask user}
          search = {flag_key}

        IF matching flag exists:
          POST comment: "Flag `{flag_key}` already exists: {url}"
          SKIP creation
          CONTINUE to next opportunity

     c. DETERMINE project key:
        IF project_key can be inferred from context:
          USE inferred project_key
        ELSE:
          ASK user: "Which LaunchDarkly project should I create this flag in?"
          WAIT for response
          USE provided project_key

     d. DELEGATE flag creation:
        INVOKE flag-creator sub-agent with:
          project_key, flag_key, flag_name, description, tags, flag_type, temporary

        RECEIVE result: { success, flag_key, flag_url, environments, error }

     e. HANDLE result:
        IF success:
          COLLECT flag details for summary
        ELSE:
          COLLECT error for summary

  4. POST summary comment on PR/issue:
     FOR each created flag:
       - Flag name and key
       - Environments it was created in
       - Direct link to LaunchDarkly UI
       - Suggested next steps ("wrap lines X–Y in your flag evaluation")

     FOR each error:
       - Flag key and error message
```

## Flag Key Normalization

```
FUNCTION normalize(input):
  result = input
  result = lowercase(result)
  result = replace(result, /[^a-z0-9-]/g, "-")  // replace non-alphanumeric with hyphens
  result = replace(result, /--+/g, "-")           // collapse multiple hyphens
  result = trim(result, "-")                      // remove leading/trailing hyphens
  result = remove_issue_numbers(result)            // strip #123 patterns
  RETURN result
```
