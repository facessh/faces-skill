# Contributing & Bug Reports

## Filing an issue

If a command fails unexpectedly — especially after updating the CLI — file a
report so the team can fix it:

```bash
gh issue create --repo faces-sh/faces-skill \
  --title "Bug: <one-line summary>" \
  --label "agent-report" \
  --body "$(cat <<'ISSUE'
## What happened
<what you were trying to do and what went wrong>

## CLI version
$(faces --version 2>&1)

## Commands that failed
```
<paste the exact command and error output>
```

## Expected behavior
<what should have happened>

## Context
- Skill: /face | /faceteam | /manyface | /faces
- OS: $(uname -s) $(uname -m)
- Node: $(node --version 2>&1)
ISSUE
)"
```

If `gh` is not installed or not authenticated, open an issue manually at:
https://github.com/faces-sh/faces-skill/issues

## What to report

- CLI commands that fail with unexpected errors
- Skill instructions that produce wrong results
- Missing or incorrect documentation
- Workflows that could be simpler

## What NOT to report

- Faces platform outages (contact support at faces.sh)
- Feature requests for the CLI (file at the CLI repo, not the skill repo)
- Questions about how to use Faces (use `/faces` or read the docs at docs.faces.sh)
