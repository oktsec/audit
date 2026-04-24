# audit - Security audit for AI-built code

Run `/audit` in any project. Get a graded security report with exact fixes in under 2 minutes.

Built for code written with Claude Code, Cursor, Copilot, Windsurf, and other AI coding tools. Covers OWASP Top 10 with 130+ detection patterns across 19 security categories.

```bash
npx skills add oktsec/audit
```

Works with Claude Code, Codex, Gemini CLI, Amp, and [40+ other agents](https://skills.sh).

## What it does

Most AI-generated code ships with the same security gaps: hardcoded API keys, missing auth on API routes, SQL injection via string interpolation, CORS wildcards, no rate limiting. This skill finds them before your users do.

```
## Security Audit

**Project:** my-saas
**Stack:** Next.js 14 + Supabase + Stripe + Vercel
**Scanned:** 142 files across 23 directories

### Score: D

2 critical findings require immediate attention.

**1. Supabase service_role key in client bundle** `CRITICAL`
📍 `.env.local:7`
Service role bypasses all Row Level Security. Any user can read/write all database tables.

**2. Stripe webhook without signature verification** `HIGH`
📍 `app/api/webhook/route.ts:12`
Anyone can POST fake payment events to this endpoint.

### Top 3 actions
1. Move service_role to server-side only, use anon key in client
2. Add constructEvent() with webhook secret to verify Stripe signatures
3. Enable RLS on all Supabase tables with per-user policies
```

After the report, it offers to fix issues one by one - showing what changes before applying them.

## How it works

1. **Detect** - reads your project root and identifies your stack automatically
2. **Load** - pulls only the checks relevant to your stack (progressive disclosure)
3. **Scan** - runs 130+ patterns against your code
4. **Report** - grades A through F with before/after code for every finding
5. **Fix** - optionally walks through each finding and applies the fix

A Next.js + Supabase project loads Next.js and Supabase checks. A plain Express API skips them entirely. Less noise, more relevant findings.

## What it catches

### Secrets (CRITICAL)
API keys for 16 providers (AWS, Stripe, OpenAI, Anthropic, GitHub, GitLab, Slack, SendGrid, Twilio, Mailgun, Square), private keys (RSA, EC, DSA, OpenSSH, PGP), database connection strings with embedded passwords, hardcoded secrets in code, secrets leaked in git history, missing `.gitignore` entries.

### Injection (HIGH)
SQL injection (9 patterns across JS/TS, Python, Go), XSS (7 patterns including React, Vue, jQuery, Handlebars, EJS, Django), command injection, path traversal, SSRF, unsafe deserialization (pickle, yaml.load, eval), open redirects.

### Authentication (HIGH)
Weak password hashing (MD5/SHA instead of bcrypt/argon2), insecure randomness (Math.random for tokens), JWT without expiry, algorithm none attack, missing cookie security flags, no rate limiting on login endpoints.

### Database access control (CRITICAL/HIGH)
Supabase Row Level Security enforcement, Firebase security rules validation, mass assignment via raw request body in ORM calls, MongoDB network access exposure.

### Framework-specific checks
- **Next.js**: `NEXT_PUBLIC_` env var exposure, Server Actions without auth, API routes without session checks
- **Clerk**: Secret key in client bundle, middleware without route protection
- **Supabase**: service_role key exposure, missing RLS policies, admin API in client code
- **Firebase**: Public read/write rules, blanket auth-only rules without per-document checks

### AI/LLM integration (HIGH)
API keys exposed to client bundle, prompt injection via user input interpolation in system prompts, missing rate limiting on AI endpoints.

### Payments (HIGH)
Stripe/Paddle webhook signature verification, client-side amount manipulation, missing idempotency keys.

### Infrastructure (MEDIUM-HIGH)
CORS wildcards, debug mode in production, missing security headers (HSTS, CSP, X-Frame-Options), Docker running as root, secrets in build args, unpinned base images, missing .dockerignore.

### Dependencies (MEDIUM)
npm audit / pip audit / govulncheck integration, version pinning checks (flags `*`, `latest`, `>=`).

### Data exposure (MEDIUM)
Logging passwords and request bodies, SELECT * over-fetching, full user objects in API responses, missing error monitoring.

## OWASP Top 10 coverage

| # | Category | What we check |
|---|----------|---------------|
| A01 | Broken Access Control | Supabase RLS, Firebase rules, auth middleware, Server Actions, mass assignment |
| A02 | Cryptographic Failures | Weak hashing, insecure randomness, missing HTTPS, cookie flags |
| A03 | Injection | SQL, XSS, command, SSRF, path traversal, deserialization |
| A04 | Insecure Design | Missing rate limiting, no webhook verification, no timeouts |
| A05 | Security Misconfiguration | CORS, debug mode, security headers, Docker, .gitignore |
| A06 | Vulnerable Components | npm audit, pip audit, govulncheck, version pinning |
| A07 | Auth Failures | JWT issues, cookie flags, credential storage, Clerk config |
| A08 | Data Integrity | Open redirects, unsigned webhooks, client-side amounts |
| A09 | Logging Failures | Auth event logging, error monitoring, password logging |
| A10 | SSRF | URL validation, cloud metadata access, DNS rebinding |

## Supported stacks

**Languages:** JavaScript/TypeScript, Python, Go, Rust, Ruby, PHP

**Frameworks:** Next.js, Express, Fastify, Hono, Koa, Nuxt, SvelteKit, Remix, Flask, Django, FastAPI, Gin, Echo, Chi, Fiber, Rails, Laravel

**Databases:** Prisma, Drizzle, Knex, Sequelize, TypeORM, Mongoose, SQLAlchemy, GORM, Diesel, Ent

**Auth:** NextAuth, Clerk, Lucia, Passport, Supabase Auth, Better Auth, JWT, bcrypt, argon2

**BaaS:** Supabase, Firebase, Convex, Appwrite

**AI/LLM:** OpenAI, Anthropic, Vercel AI SDK, LangChain, LlamaIndex, Google Generative AI, Cohere, Replicate

**Payments:** Stripe, Paddle, LemonSqueezy, PayPal

**Infra:** Docker, Vercel, Netlify, Fly.io, Railway, Cloudflare Workers, GitHub Actions

## Architecture

This skill uses progressive disclosure. Instead of loading all 130+ checks into context every time, it detects your stack first and loads only the relevant check files.

```
audit/
├── SKILL.md              # Orchestration (6K)
├── gotchas.md            # Known false positives
├── checks/
│   ├── _index.md         # Load map
│   ├── secrets.md        # Always loaded
│   ├── injection.md      # Always loaded
│   ├── auth.md           # Always loaded
│   ├── config.md         # Always loaded
│   ├── dependencies.md   # Always loaded
│   ├── data-exposure.md  # Always loaded
│   ├── nextjs.md         # Only if Next.js detected
│   ├── clerk.md          # Only if Clerk detected
│   ├── supabase.md       # Only if Supabase detected
│   ├── firebase.md       # Only if Firebase detected
│   ├── mongodb.md        # Only if MongoDB detected
│   ├── ai-llm.md         # Only if AI/LLM deps detected
│   ├── payments.md       # Only if Stripe/Paddle detected
│   ├── uploads.md        # Only if file uploads detected
│   ├── docker.md         # Only if Dockerfile present
│   └── integrations.md   # Only if OAuth/webhooks detected
└── templates/
    └── report.md         # Output format
```

## Install

```bash
npx skills add oktsec/audit
```

Or clone manually:

```bash
git clone https://github.com/oktsec/audit ~/.claude/skills/audit
```

Then run `/audit` in any project.

## Built by oktsec

[oktsec](https://github.com/oktsec/oktsec) builds runtime security for AI agents. This skill is the static analysis side - if you need runtime protection for agent-to-agent communication, check out the full platform.

## License

Apache-2.0
