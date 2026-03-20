# CRITICAL: Secrets in Code

Search for each pattern. Report file and line number for every match.

## API Keys

| Pattern | What |
|---------|------|
| `AKIA[0-9A-Z]{16}` | AWS access key |
| `sk_live_[a-zA-Z0-9]{24,}` | Stripe live key |
| `rk_live_[a-zA-Z0-9]{24,}` | Stripe restricted key |
| `sk-proj-[a-zA-Z0-9\-_]{20,}` | OpenAI project key |
| `sk-ant-[a-zA-Z0-9\-_]{80,}` | Anthropic key |
| `sk-[a-zA-Z0-9]{48,}` | OpenAI legacy key (long format) |
| `ghp_[a-zA-Z0-9]{36}` | GitHub PAT |
| `gho_[a-zA-Z0-9]{36}` | GitHub OAuth |
| `github_pat_[a-zA-Z0-9_]{80,}` | GitHub fine-grained PAT |
| `glpat-[a-zA-Z0-9\-_]{20,}` | GitLab PAT |
| `xoxb-[0-9]{10,}-[a-zA-Z0-9]+` | Slack bot token |
| `xoxp-[0-9]{10,}-[a-zA-Z0-9]+` | Slack user token |
| `SG\.[a-zA-Z0-9\-_]{22}\.[a-zA-Z0-9\-_]{22}` | SendGrid key |
| `sq0atp-[a-zA-Z0-9\-_]{22}` | Square access token |
| `AC[a-z0-9]{32}` | Twilio account SID (confirm: near `auth_token` or `twilio` import) |
| `key-[a-zA-Z0-9]{32}` | Mailgun key (confirm: near `mailgun` import or config) |

## Private Keys

| Pattern | What |
|---------|------|
| `-----BEGIN (RSA\|EC\|DSA\|OPENSSH) PRIVATE KEY` | Private key in code |
| `-----BEGIN PGP PRIVATE KEY` | PGP private key |

## Connection Strings with Credentials

| Pattern | What |
|---------|------|
| `(postgres\|postgresql\|mysql\|mongodb\+srv\|mongodb):\/\/[^:\s]+:[^@\s]+@` | Database connection string with embedded password |
| `redis:\/\/:[^@\s]+@` | Redis with password |
| `amqps?:\/\/[^:\s]+:[^@\s]+@` | RabbitMQ with password |

## Hardcoded Secrets in Assignments

| Pattern | What |
|---------|------|
| `(password\|passwd\|secret\|api_key\|apiKey\|api_secret\|token\|auth_token)\s*[:=]\s*['"][A-Za-z0-9\-_./+]{12,}['"]` | Hardcoded secret value (12+ chars, not a placeholder) |

## Context Rules - Adjust Severity

- In `*.test.*`, `*.spec.*`, `__tests__/`, `test/`, `fixtures/`, `testdata/`, `mock/`: downgrade to INFO
- In `*.example`, `*.sample`, `*.template`, `README*`, `docs/`: downgrade to INFO
- If the value contains `xxx`, `your-`, `TODO`, `CHANGE`, `example`, `placeholder`, `dummy`, `test`, `fake`: downgrade to INFO
- In `.env` that IS in `.gitignore`: downgrade to INFO (correctly handled)
- In `.env` that is NOT in `.gitignore`: keep CRITICAL
- In committed source code with real-looking values: keep CRITICAL

## .gitignore Validation

Read `.gitignore`. Verify these entries exist:
```
.env
.env.local
.env.*.local
*.pem
*.key
```
If `.env` is missing from `.gitignore`: CRITICAL finding on its own.

## Git History Check

```bash
git log --all --oneline -- '.env' '.env.local' '*.pem' '*.key' 2>/dev/null | head -10
```
If any results: secrets may be in git history even if currently gitignored. CRITICAL. Tell the user they need to rotate those credentials - removing a file from git does not remove it from history.
