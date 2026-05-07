# Test Scripts — Flag Creator Sub-Agent

> Note: The flag-creator sub-agent interacts exclusively with the LaunchDarkly MCP server. Full testing requires a live LaunchDarkly environment. Scripts below validate configuration.

## TS-2.2.1: Validate flag-creator agent front matter

```bash
#!/usr/bin/env bash
set -euo pipefail

AGENT="agents/flag-creator.agent.md"
echo "=== Validating $AGENT ==="

if [ ! -f "$AGENT" ]; then
  echo "FAIL: $AGENT not found"
  exit 1
fi

frontmatter=$(sed -n '/^---$/,/^---$/p' "$AGENT")

# Check name
if echo "$frontmatter" | grep -q "name: flag-creator"; then
  echo "PASS: Agent name is correct"
else
  echo "FAIL: Agent name is incorrect"
  exit 1
fi

# Check tools is empty (no file system tools)
if echo "$frontmatter" | grep -q 'tools: \[\]'; then
  echo "PASS: tools is empty (correct for sub-agent)"
else
  echo "FAIL: tools should be empty for flag-creator"
  exit 1
fi

# Check MCP server
if echo "$frontmatter" | grep -q "oidc: true"; then
  echo "PASS: OIDC is enabled"
else
  echo "FAIL: OIDC not enabled"
  exit 1
fi

echo "All flag-creator checks passed."
```

## TS-2.2.2: Validate agent body has required sections

```bash
#!/usr/bin/env bash
set -euo pipefail

AGENT="agents/flag-creator.agent.md"
echo "=== Validating $AGENT body ==="

REQUIRED=("Input you expect" "What you do" "Error handling")

for section in "${REQUIRED[@]}"; do
  if grep -qi "$section" "$AGENT"; then
    echo "PASS: '$section' section found"
  else
    echo "FAIL: '$section' section not found"
    exit 1
  fi
done

echo "All body checks passed."
```

## TS-2.2.3: Integration test scenarios (manual)

```markdown
### Scenario A: Create a new boolean flag
1. Invoke flag-creator with project_key="default", flag_key="test-flag-123"
2. Verify: create_feature_flag MCP tool is called
3. Verify: get_feature_flag MCP tool is called to verify
4. Verify: Result contains success=true, flag_url, environments

### Scenario B: Attempt to create duplicate flag
1. Create "test-flag-dup" manually in LaunchDarkly
2. Invoke flag-creator with flag_key="test-flag-dup"
3. Verify: Result contains success=false, error about existing flag

### Scenario C: Invalid project key
1. Invoke flag-creator with project_key="nonexistent-project"
2. Verify: Result contains success=false, clear error about invalid project
```
