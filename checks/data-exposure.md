# MEDIUM: Data Exposure

## Logging Sensitive Data

| Pattern | Issue |
|---------|-------|
| `console\.log\(.*req\.body` | Logging full request body (may contain passwords) |
| `console\.log\(.*request\.body` | Logging full request body (Python/Express) |
| `console\.log\(.*password` | Explicitly logging passwords |
| `print\(.*password` | Explicitly logging passwords (Python) |
| `logger\.\w+\(.*password` | Logging passwords via logger |

## Over-Fetching

| Pattern | Issue |
|---------|-------|
| `SELECT\s+\*` | Over-fetching (skip matches in migrations, seeds, and test files) |
| `JSON\.stringify\(user\)` | May serialize password hash, internal fields to response |
| `res\.json\(user\)` | Full user object in API response (should select specific fields) |

## Logging and Monitoring

Check if the app has any security-relevant logging:
- Search for login/auth event logging. If auth exists but zero log statements on failed logins: MEDIUM (can't detect brute force)
- Search for error monitoring: `sentry`, `bugsnag`, `datadog`, `newrelic`, `logtail` in deps. If none and the app is production-ready: MEDIUM
- If error handlers use `console.log` only (no structured logging, no external service): note as improvement
