# Pseudo Logic — List Flags Skill

```
FUNCTION list_flags(project_key, search?, tag?, limit?):

  1. CALL MCP list_feature_flags:
       project_key = project_key
       search      = search (if provided)
       tag         = tag (if provided)
       limit       = limit (default: 20)

  2. IF results are empty:
       RETURN "No existing flags found matching your criteria."

  3. FORMAT results:
       output = "Existing flags in project '{project_key}':\n"
       FOR each flag in results:
         output += "- `{flag.key}` — {flag.name} (created {flag.creation_date})\n"

  4. IF a flag matches the intended key (if search was provided):
       HIGHLIGHT: "⚠ Flag `{search}` already exists!"

  5. RETURN output
```
