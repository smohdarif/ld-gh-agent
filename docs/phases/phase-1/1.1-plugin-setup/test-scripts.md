# Test Scripts — Plugin Setup & Configuration

## TS-1.1.1: Validate plugin.json structure

```bash
#!/usr/bin/env bash
# Verify plugin.json is valid JSON and has required fields

set -euo pipefail

FILE="plugin.json"

echo "=== Validating $FILE ==="

# Check file exists
if [ ! -f "$FILE" ]; then
  echo "FAIL: $FILE not found"
  exit 1
fi

# Check valid JSON
if ! jq empty "$FILE" 2>/dev/null; then
  echo "FAIL: $FILE is not valid JSON"
  exit 1
fi

# Check required fields
for field in name description version agents skills hooks; do
  if ! jq -e ".$field" "$FILE" >/dev/null 2>&1; then
    echo "FAIL: Missing required field '$field'"
    exit 1
  fi
done

echo "PASS: $FILE is valid with all required fields"
```

## TS-1.1.2: Validate agent files exist

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== Validating agent files ==="

AGENTS=("agents/main.agent.md" "agents/flag-creator.agent.md")

for agent in "${AGENTS[@]}"; do
  if [ -f "$agent" ]; then
    echo "PASS: $agent exists"
  else
    echo "FAIL: $agent not found"
    exit 1
  fi
done
```

## TS-1.1.3: Validate skill files exist

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== Validating skill files ==="

SKILLS=("skills/create-flag/SKILL.md" "skills/list-flags/SKILL.md" "skills/suggest-flags/SKILL.md")

for skill in "${SKILLS[@]}"; do
  if [ -f "$skill" ]; then
    echo "PASS: $skill exists"
  else
    echo "FAIL: $skill not found"
    exit 1
  fi
done
```

## TS-1.1.4: Test pre-run hook with OIDC token

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== Testing validate-ld-context.sh with OIDC token ==="

export GITHUB_COPILOT_OIDC_MCP_TOKEN="test-token"
unset LD_API_KEY 2>/dev/null || true

if bash scripts/validate-ld-context.sh; then
  echo "PASS: Script exits 0 with OIDC token"
else
  echo "FAIL: Script should exit 0 with OIDC token"
  exit 1
fi
```

## TS-1.1.5: Test pre-run hook with API key

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== Testing validate-ld-context.sh with API key ==="

unset GITHUB_COPILOT_OIDC_MCP_TOKEN 2>/dev/null || true
export LD_API_KEY="api-test-key"

if bash scripts/validate-ld-context.sh; then
  echo "PASS: Script exits 0 with API key"
else
  echo "FAIL: Script should exit 0 with API key"
  exit 1
fi
```

## TS-1.1.6: Test pre-run hook with no auth

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== Testing validate-ld-context.sh with no auth ==="

unset GITHUB_COPILOT_OIDC_MCP_TOKEN 2>/dev/null || true
unset LD_API_KEY 2>/dev/null || true

if bash scripts/validate-ld-context.sh 2>/dev/null; then
  echo "FAIL: Script should exit non-zero with no auth"
  exit 1
else
  echo "PASS: Script exits non-zero with no auth"
fi
```
