# audit

Security audit for AI-built projects. Covers OWASP Top 10.

Auto-detects your stack, loads only the relevant checks, and produces a graded report with exact code fixes. Built by [oktsec](https://github.com/oktsec).

## Install

```bash
npx skills add oktsec/audit
```

Or manually:

```bash
git clone https://github.com/oktsec/audit ~/.claude/skills/audit
```

## Usage

Run `/audit` in any project. No setup needed.

## What it detects

- **Secrets**: API keys (16 providers), private keys, connection strings, hardcoded credentials, git history leaks
- **Injection**: SQL injection, XSS, command injection, SSRF, path traversal, deserialization, open redirects
- **Auth**: Weak password hashing, insecure randomness, JWT misconfigurations, cookie flags, missing rate limiting
- **Database**: Supabase RLS, Firebase security rules, mass assignment
- **Config**: CORS wildcards, debug mode, missing security headers, exposed internals
- **AI/LLM**: API key exposure, prompt injection, missing rate limits on AI endpoints
- **Payments**: Webhook signature verification, client-side amounts, idempotency
- **Infra**: Docker as root, secrets in build args, unpinned images

## How it works

1. **Detect** - reads your project and identifies the stack (framework, database, auth, payments, AI, infra)
2. **Load** - pulls only the relevant check files (Next.js checks only run if Next.js is detected)
3. **Scan** - runs 130+ patterns against your code using Grep and Glob
4. **Report** - grades your project (A-F) with findings, before/after code, and top 3 actions
5. **Fix** - optionally walks through each finding and applies fixes

## Example output

```
## Security Audit

**Project:** my-saas
**Stack:** Next.js 14 + Supabase + Stripe + Vercel
**Scanned:** 142 files across 23 directories
**Date:** 2026-03-20

### Score: D

2 critical findings require immediate attention.

**1. Supabase service_role key in client bundle** `CRITICAL`
📍 `.env.local:7`
Service role bypasses all Row Level Security. Any user can read/write all database tables.

**2. Stripe webhook without signature verification** `HIGH`
📍 `app/api/webhook/route.ts:12`
Anyone can POST fake payment events to this endpoint.
```

## OWASP Top 10 Coverage

| OWASP | Category | Checks |
|-------|----------|--------|
| A01 | Broken Access Control | RLS, Firebase rules, auth middleware, mass assignment |
| A02 | Cryptographic Failures | Weak hashing, insecure randomness, missing HTTPS |
| A03 | Injection | SQL, XSS, command, SSRF, path traversal, deserialization |
| A04 | Insecure Design | Missing rate limiting, no webhook verification |
| A05 | Security Misconfiguration | CORS, debug mode, security headers, Docker |
| A06 | Vulnerable Components | npm audit, pip audit, govulncheck, version pinning |
| A07 | Auth Failures | JWT issues, cookie flags, credential storage |
| A08 | Data Integrity | Open redirects, unsigned webhooks |
| A09 | Logging Failures | Missing auth logging, no error monitoring |
| A10 | SSRF | URL validation, cloud metadata access |

## Architecture

This skill uses progressive disclosure - only loads the checks relevant to your stack:

```
audit/
├── SKILL.md              # Core orchestration (detect → load → scan → report → fix)
├── gotchas.md            # Known false positives and common mistakes
├── checks/
│   ├── _index.md         # Which checks to load based on detected stack
│   ├── secrets.md        # CORE: always loaded
│   ├── injection.md      # CORE: always loaded
│   ├── auth.md           # CORE: always loaded
│   ├── config.md         # CORE: always loaded
│   ├── dependencies.md   # CORE: always loaded
│   ├── data-exposure.md  # CORE: always loaded
│   ├── nextjs.md         # CONDITIONAL: Next.js only
│   ├── clerk.md          # CONDITIONAL: Clerk only
│   ├── supabase.md       # CONDITIONAL: Supabase only
│   ├── firebase.md       # CONDITIONAL: Firebase only
│   ├── mongodb.md        # CONDITIONAL: MongoDB only
│   ├── ai-llm.md         # CONDITIONAL: AI/LLM only
│   ├── payments.md       # CONDITIONAL: Stripe/Paddle only
│   ├── uploads.md        # CONDITIONAL: File uploads only
│   ├── docker.md         # CONDITIONAL: Docker only
│   └── integrations.md   # CONDITIONAL: OAuth/webhooks only
└── templates/
    └── report.md         # Output format template
```

## License

Apache-2.0
