# Test Scripts — Main Agent (Orchestrator)

> Note: The main agent is LLM-driven, so testing is primarily scenario-based (manual or integration tests in a live GitHub Copilot Extensions environment). The scripts below validate the agent's configuration and structure.

## TS-2.1.1: Validate main agent front matter

```bash
#!/usr/bin/env bash
set -euo pipefail

AGENT="agents/main.agent.md"
echo "=== Validating $AGENT ==="

# Check file exists
if [ ! -f "$AGENT" ]; then
  echo "FAIL: $AGENT not found"
  exit 1
fi

# Extract front matter
frontmatter=$(sed -n '/^---$/,/^---$/p' "$AGENT")

# Check required fields
for field in "name:" "description:" "tools:" "mcp-servers:"; do
  if echo "$frontmatter" | grep -q "$field"; then
    echo "PASS: '$field' found in front matter"
  else
    echo "FAIL: '$field' not found in front matter"
    exit 1
  fi
done

# Check agent name
if echo "$frontmatter" | grep -q "name: launchdarkly-agent"; then
  echo "PASS: Agent name is correct"
else
  echo "FAIL: Agent name is incorrect"
  exit 1
fi

echo "All main agent checks passed."
```

## TS-2.1.2: Validate agent body contains required instructions

```bash
#!/usr/bin/env bash
set -euo pipefail

AGENT="agents/main.agent.md"
echo "=== Validating $AGENT body content ==="

# Check for key instruction sections
REQUIRED_SECTIONS=(
  "Core responsibilities"
  "Flag naming conventions"
  "Escalation"
  "Detect flag opportunities"
  "Avoid duplicates"
  "Report back"
)

for section in "${REQUIRED_SECTIONS[@]}"; do
  if grep -qi "$section" "$AGENT"; then
    echo "PASS: Section '$section' found"
  else
    echo "FAIL: Section '$section' not found"
    exit 1
  fi
done

echo "All body content checks passed."
```

## TS-2.1.3: Integration test scenarios (manual)

```markdown
### Scenario A: PR with new feature
1. Create a PR that adds a new UI component with conditional rendering
2. @-mention the agent in a PR comment
3. Verify: Agent analyzes the diff and suggests a flag
4. Verify: Agent checks for duplicates via list-flags
5. Verify: Agent creates the flag and posts a summary comment

### Scenario B: Issue requesting gradual rollout
1. Create an issue titled "Roll out new checkout to 10% of users"
2. @-mention the agent in the issue
3. Verify: Agent identifies this as a flag opportunity
4. Verify: Agent asks for the project key if not determinable

### Scenario C: PR with no flag opportunities
1. Create a PR that only updates documentation
2. @-mention the agent
3. Verify: Agent responds "No feature flag opportunities detected"

### Scenario D: Duplicate flag
1. Create a flag `new-checkout-flow` manually in LaunchDarkly
2. Create a PR on branch `feature/new-checkout-flow`
3. @-mention the agent
4. Verify: Agent detects the existing flag and skips creation
```
