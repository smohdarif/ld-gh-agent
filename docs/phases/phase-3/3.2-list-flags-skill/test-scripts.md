# Test Scripts — List Flags Skill

## TS-3.2.1: Validate skill file structure

```bash
#!/usr/bin/env bash
set -euo pipefail

SKILL="skills/list-flags/SKILL.md"
echo "=== Validating $SKILL ==="

if [ ! -f "$SKILL" ]; then
  echo "FAIL: $SKILL not found"
  exit 1
fi

frontmatter=$(sed -n '/^---$/,/^---$/p' "$SKILL")

if echo "$frontmatter" | grep -q "name: list-flags"; then
  echo "PASS: Skill name is correct"
else
  echo "FAIL: Skill name incorrect"
  exit 1
fi

for section in "Parameters" "Steps" "Output format"; do
  if grep -qi "$section" "$SKILL"; then
    echo "PASS: '$section' section found"
  else
    echo "FAIL: '$section' section not found"
    exit 1
  fi
done

echo "All list-flags skill checks passed."
```
