# Report Template

Output this exact structure. Be specific - reference actual files and line numbers.

```
## Security Audit

**Project:** [name from package.json/go.mod/pyproject.toml]
**Stack:** [detected stack]
**Scanned:** [N files across M directories]
**Date:** [today]

### Score: [letter]

[One sentence explaining the score]
```

Then list findings grouped by severity (CRITICAL first, then HIGH, MEDIUM, INFO). For EACH finding:

```
**[N]. [Title]** `[SEVERITY]`
📍 `file/path.js:42`
[One sentence: what it is and what an attacker can do with it]

Before:
‎```[lang]
[insecure code from their file - REDACT any secret values: show first 4 chars + ****]
‎```

After:
‎```[lang]
[secure replacement - use environment variable references, never actual secret values]
‎```
```

If the fix needs a package install, include the install command before the code fix.

After all findings, add:

```
### What's solid
- [2-3 specific things the codebase does well. Be genuine, not filler.]

### Top 3 actions
1. [Highest impact fix. One sentence + the command or code change.]
2. [Second priority.]
3. [Third priority.]
```
