# Test Scripts — OIDC Authentication

> Note: OIDC token exchange is handled by the GitHub platform, so these tests focus on validating the configuration and the pre-run hook. Full end-to-end OIDC testing requires a live GitHub Copilot Extensions environment.

## TS-1.2.1: Validate OIDC config in agent front matter

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== Validating OIDC config in agent front matter ==="

for agent in agents/main.agent.md agents/flag-creator.agent.md; do
  echo "Checking $agent..."

  # Extract YAML front matter (between --- markers)
  frontmatter=$(sed -n '/^---$/,/^---$/p' "$agent")

  # Check oidc: true is present
  if echo "$frontmatter" | grep -q "oidc: true"; then
    echo "  PASS: oidc: true found"
  else
    echo "  FAIL: oidc: true not found"
    exit 1
  fi

  # Check MCP server URL is present
  if echo "$frontmatter" | grep -q "url: https://mcp.launchdarkly.com"; then
    echo "  PASS: LaunchDarkly MCP URL found"
  else
    echo "  FAIL: LaunchDarkly MCP URL not found"
    exit 1
  fi

  # Check type is http
  if echo "$frontmatter" | grep -q "type: http"; then
    echo "  PASS: type: http found"
  else
    echo "  FAIL: type: http not found"
    exit 1
  fi
done

echo "All OIDC config checks passed."
```

## TS-1.2.2: Pre-run hook — all auth scenarios

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT="scripts/validate-ld-context.sh"
PASS=0
FAIL=0

echo "=== Testing $SCRIPT auth scenarios ==="

# Scenario 1: OIDC token only
echo "--- Scenario 1: OIDC token only ---"
(
  export GITHUB_COPILOT_OIDC_MCP_TOKEN="test-oidc-token"
  unset LD_API_KEY 2>/dev/null || true
  if bash "$SCRIPT"; then
    echo "PASS"; exit 0
  else
    echo "FAIL"; exit 1
  fi
) && ((PASS++)) || ((FAIL++))

# Scenario 2: API key only
echo "--- Scenario 2: API key only ---"
(
  unset GITHUB_COPILOT_OIDC_MCP_TOKEN 2>/dev/null || true
  export LD_API_KEY="api-test-key"
  if bash "$SCRIPT"; then
    echo "PASS"; exit 0
  else
    echo "FAIL"; exit 1
  fi
) && ((PASS++)) || ((FAIL++))

# Scenario 3: Both present
echo "--- Scenario 3: Both present ---"
(
  export GITHUB_COPILOT_OIDC_MCP_TOKEN="test-oidc-token"
  export LD_API_KEY="api-test-key"
  if bash "$SCRIPT"; then
    echo "PASS"; exit 0
  else
    echo "FAIL"; exit 1
  fi
) && ((PASS++)) || ((FAIL++))

# Scenario 4: Neither present
echo "--- Scenario 4: Neither present ---"
(
  unset GITHUB_COPILOT_OIDC_MCP_TOKEN 2>/dev/null || true
  unset LD_API_KEY 2>/dev/null || true
  if bash "$SCRIPT" 2>/dev/null; then
    echo "FAIL (should have exited non-zero)"; exit 1
  else
    echo "PASS (correctly rejected)"; exit 0
  fi
) && ((PASS++)) || ((FAIL++))

echo ""
echo "=== Results: $PASS passed, $FAIL failed ==="
[ "$FAIL" -eq 0 ] || exit 1
```
