# Test Scripts — Create Flag Skill

## TS-3.1.1: Validate skill file structure

```bash
#!/usr/bin/env bash
set -euo pipefail

SKILL="skills/create-flag/SKILL.md"
echo "=== Validating $SKILL ==="

if [ ! -f "$SKILL" ]; then
  echo "FAIL: $SKILL not found"
  exit 1
fi

# Check front matter fields
frontmatter=$(sed -n '/^---$/,/^---$/p' "$SKILL")

if echo "$frontmatter" | grep -q "name: create-flag"; then
  echo "PASS: Skill name is correct"
else
  echo "FAIL: Skill name incorrect"
  exit 1
fi

# Check required sections in body
for section in "Required parameters" "Optional parameters" "Steps" "Output format"; do
  if grep -qi "$section" "$SKILL"; then
    echo "PASS: '$section' section found"
  else
    echo "FAIL: '$section' section not found"
    exit 1
  fi
done

echo "All create-flag skill checks passed."
```
