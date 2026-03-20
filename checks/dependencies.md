# MEDIUM: Dependencies

## Run Audit

Only if the tool is available:

```bash
# Node.js (human-readable summary, not JSON)
npm audit --omit=dev 2>/dev/null

# Python
pip audit 2>/dev/null || pip-audit 2>/dev/null

# Go
govulncheck ./... 2>/dev/null
```

Report summary: X critical, Y high, Z moderate vulnerabilities.

## Version Pinning

Check `package.json` dependencies (not devDependencies) for:

| Pattern | Issue |
|---------|-------|
| `"*"` | Completely unpinned - any version |
| `"latest"` | Unpinned - resolves at install time |
| `">="` | Open-ended range - no upper bound |

Note: `^` and `~` in npm are normal (semver ranges). Only flag `*`, `latest`, and `>=`.

For `requirements.txt`: lines without `==` are unpinned. Flag them.
