# Test Scripts — Invocation Flow

> Note: The invocation flow is orchestrated by the GitHub Copilot Extensions platform, so full end-to-end testing requires a live environment. The scripts below validate the local configuration that supports the flow.

## TS-2.0.1: Validate all components are discoverable

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== Validating plugin component discovery ==="

ERRORS=0

# Check plugin.json
if [ -f "plugin.json" ]; then
  echo "PASS: plugin.json exists"
else
  echo "FAIL: plugin.json not found"
  ((ERRORS++))
fi

# Check agents directory
AGENTS=("agents/main.agent.md" "agents/flag-creator.agent.md")
for agent in "${AGENTS[@]}"; do
  if [ -f "$agent" ]; then
    echo "PASS: $agent exists"
  else
    echo "FAIL: $agent not found"
    ((ERRORS++))
  fi
done

# Check skills
SKILLS=("skills/create-flag/SKILL.md" "skills/list-flags/SKILL.md" "skills/suggest-flags/SKILL.md")
for skill in "${SKILLS[@]}"; do
  if [ -f "$skill" ]; then
    echo "PASS: $skill exists"
  else
    echo "FAIL: $skill not found"
    ((ERRORS++))
  fi
done

# Check hooks
if [ -f "hooks.json" ]; then
  echo "PASS: hooks.json exists"
else
  echo "FAIL: hooks.json not found"
  ((ERRORS++))
fi

# Check validation script
if [ -f "scripts/validate-ld-context.sh" ]; then
  echo "PASS: validate-ld-context.sh exists"
  if [ -x "scripts/validate-ld-context.sh" ] || head -1 "scripts/validate-ld-context.sh" | grep -q "bash"; then
    echo "PASS: validate-ld-context.sh is executable or has bash shebang"
  else
    echo "WARN: validate-ld-context.sh may not be executable"
  fi
else
  echo "FAIL: validate-ld-context.sh not found"
  ((ERRORS++))
fi

echo ""
if [ "$ERRORS" -eq 0 ]; then
  echo "=== All components discoverable ==="
else
  echo "=== $ERRORS components missing ==="
  exit 1
fi
```

## TS-2.0.2: Validate agent-to-agent consistency

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== Validating agent consistency ==="

MAIN="agents/main.agent.md"
SUB="agents/flag-creator.agent.md"

# Both agents should connect to the same MCP server
MAIN_URL=$(grep "url:" "$MAIN" | head -1 | awk '{print $2}')
SUB_URL=$(grep "url:" "$SUB" | head -1 | awk '{print $2}')

if [ "$MAIN_URL" = "$SUB_URL" ]; then
  echo "PASS: Both agents connect to same MCP server: $MAIN_URL"
else
  echo "FAIL: MCP server URLs differ: main=$MAIN_URL sub=$SUB_URL"
  exit 1
fi

# Both agents should use OIDC
for agent in "$MAIN" "$SUB"; do
  if grep -q "oidc: true" "$agent"; then
    echo "PASS: $agent has oidc: true"
  else
    echo "FAIL: $agent missing oidc: true"
    exit 1
  fi
done

# Main agent should have tools, sub-agent should not
if grep -q 'tools: \["view", "edit"\]' "$MAIN"; then
  echo "PASS: Main agent has view/edit tools"
else
  echo "FAIL: Main agent missing view/edit tools"
  exit 1
fi

if grep -q 'tools: \[\]' "$SUB"; then
  echo "PASS: Sub-agent has no file system tools (correct)"
else
  echo "FAIL: Sub-agent should have empty tools"
  exit 1
fi

echo "=== All consistency checks passed ==="
```

## TS-2.0.3: End-to-end integration test (manual)

```markdown
### Full Flow Test

1. Install plugin in a test repository
2. Create a PR with a new feature component that includes:
   - A new function with conditional logic
   - A comment: `// TODO: gate this behind a feature flag`
3. Post a comment: `@launchdarkly-agent please analyze this PR`
4. Verify the following sequence:
   a. Agent responds (platform detected @-mention) ✓
   b. Agent identifies the flag opportunity ✓
   c. Agent checks for duplicates (list-flags) ✓
   d. Agent creates the flag (flag-creator sub-agent) ✓
   e. Agent posts summary comment with:
      - Flag name and key ✓
      - Dashboard URL ✓
      - Environments ✓
      - Suggested code lines to wrap ✓
5. Verify in LaunchDarkly dashboard:
   - Flag exists with correct key, name, description ✓
   - Flag is tagged with repo and PR number ✓
   - Flag is OFF in all environments ✓
6. Verify cleanup:
   - OIDC token is no longer valid (optional — hard to verify externally)
```
