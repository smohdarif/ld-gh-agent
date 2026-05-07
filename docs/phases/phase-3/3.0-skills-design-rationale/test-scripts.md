# Test Scripts — Skills Design Rationale

## TS-3.0.1: Validate all skills are discoverable

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== Validating skill discovery ==="

EXPECTED_SKILLS=("create-flag" "list-flags" "suggest-flags")
ERRORS=0

for skill in "${EXPECTED_SKILLS[@]}"; do
  SKILL_FILE="skills/$skill/SKILL.md"
  if [ -f "$SKILL_FILE" ]; then
    echo "PASS: $SKILL_FILE exists"

    # Verify front matter has name matching directory
    if grep -q "name: $skill" "$SKILL_FILE"; then
      echo "  PASS: name matches directory"
    else
      echo "  FAIL: name in front matter doesn't match directory name '$skill'"
      ((ERRORS++))
    fi
  else
    echo "FAIL: $SKILL_FILE not found"
    ((ERRORS++))
  fi
done

echo ""
if [ "$ERRORS" -eq 0 ]; then
  echo "=== All skills discoverable ==="
else
  echo "=== $ERRORS issues found ==="
  exit 1
fi
```

## TS-3.0.2: Validate skill front matter consistency

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== Validating skill front matter ==="

for skill_dir in skills/*/; do
  skill_file="${skill_dir}SKILL.md"
  if [ ! -f "$skill_file" ]; then
    echo "FAIL: $skill_file not found"
    exit 1
  fi

  echo "Checking $skill_file..."

  # Extract front matter
  frontmatter=$(sed -n '/^---$/,/^---$/p' "$skill_file")

  # Check required fields
  for field in "name:" "description:"; do
    if echo "$frontmatter" | grep -q "$field"; then
      echo "  PASS: '$field' present"
    else
      echo "  FAIL: '$field' missing"
      exit 1
    fi
  done
done

echo "All skill front matter checks passed."
```

## TS-3.0.3: Check for overlap between flag-creator agent and create-flag skill

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== Checking agent/skill overlap ==="

AGENT="agents/flag-creator.agent.md"
SKILL="skills/create-flag/SKILL.md"

echo "Key phrases in both files:"
echo ""

# Check for shared MCP tool references
for tool in "create_feature_flag" "get_feature_flag"; do
  in_agent=$(grep -c "$tool" "$AGENT" || true)
  in_skill=$(grep -c "$tool" "$SKILL" || true)
  echo "  '$tool':"
  echo "    In agent: $in_agent occurrence(s)"
  echo "    In skill: $in_skill occurrence(s)"
  if [ "$in_agent" -gt 0 ] && [ "$in_skill" -gt 0 ]; then
    echo "    → OVERLAP detected (expected — see design rationale)"
  fi
  echo ""
done

echo "Note: Overlap between flag-creator agent and create-flag skill is intentional."
echo "See docs/phases/phase-3/3.0-skills-design-rationale/ for explanation."
```

## TS-3.0.4: Validate plugin.json references skills directory

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== Validating plugin.json skills reference ==="

if jq -e '.skills' plugin.json >/dev/null 2>&1; then
  SKILLS_DIR=$(jq -r '.skills' plugin.json)
  echo "PASS: plugin.json references skills at '$SKILLS_DIR'"

  if [ -d "$SKILLS_DIR" ]; then
    SKILL_COUNT=$(find "$SKILLS_DIR" -name "SKILL.md" | wc -l | tr -d ' ')
    echo "PASS: Found $SKILL_COUNT skill(s) in $SKILLS_DIR"
  else
    echo "FAIL: Skills directory '$SKILLS_DIR' does not exist"
    exit 1
  fi
else
  echo "FAIL: plugin.json has no 'skills' field"
  exit 1
fi
```
