---
name: audit
description: >
  Security audit for AI-built code. Runs 130+ checks across OWASP Top 10, auto-detects
  your stack, and produces a graded report (A-F) with exact code fixes. Covers secrets,
  injection, auth, database access control, AI/LLM security, payments, and infrastructure.
metadata:
  author: oktsec
  version: 3.0.0
  license: Apache-2.0
---

# `/audit` - Security audit for AI-built code

Run `/audit` in any project to get a graded security report (A-F) with exact fixes. 130+ checks across OWASP Top 10.

Built for code written with Claude Code, Cursor, Copilot, Windsurf, and other AI coding tools.

```
npx skills add oktsec/audit
```

## When to use this skill

- You want to review your code for security issues ("is this secure?", "security check", "audit my project")
- You built something with AI and want to check for vulnerabilities before deploying
- You need to verify OWASP Top 10 compliance
- You want to find hardcoded secrets, injection vulnerabilities, or auth misconfigurations
- You are deploying to production for the first time and want a security review

## What it catches

Hardcoded API keys (16 providers), SQL injection, XSS, command injection, SSRF, path traversal, weak password hashing, JWT misconfigurations, missing rate limiting, CORS wildcards, Supabase RLS gaps, Firebase rule misconfigurations, Stripe webhook verification, prompt injection, Docker misconfigurations, and more.

## How it works

Detects your stack automatically, loads only the relevant checks (Next.js checks only run if Next.js is detected), scans with 130+ patterns, then produces a graded report with before/after code for every finding. Optionally fixes issues one by one.

## Supported stacks

Next.js, Express, Fastify, Django, Flask, FastAPI, Gin, Rails, Laravel, Prisma, Supabase, Firebase, Clerk, Stripe, OpenAI, Anthropic, Docker, and 30+ more.

---

No questions. No setup. Read the project and review it.

## Instructions

### Phase 1: Detect

Read the project root. Detect everything automatically. Do NOT ask the user questions.

**Stack detection** - check for these files:
- `package.json` → Node.js. Read it: check for next, express, fastify, hono, koa, nuxt, svelte, remix
- `requirements.txt` / `pyproject.toml` / `Pipfile` → Python. Check for flask, django, fastapi, starlette
- `go.mod` → Go. Check for gin, echo, chi, fiber, net/http
- `Cargo.toml` → Rust
- `Gemfile` → Ruby / Rails
- `composer.json` → PHP / Laravel

**Database** - check deps and imports for: prisma, drizzle, knex, sequelize, typeorm, mongoose, sqlalchemy, gorm, diesel, ent

**BaaS** - check for: `@supabase/supabase-js`, `firebase`, `firebase-admin`, `convex`, `appwrite`. These have their own auth and access control rules that need separate checks

**Auth** - check deps for: next-auth, @auth/core, `@clerk/nextjs`, `@clerk/clerk-sdk-node`, `better-auth`, supabase, passport, lucia, jwt, bcrypt, argon2

**OAuth providers** - check for: passport-google, passport-github, @auth/google, next-auth providers, oauth2 client libraries

**AI/LLM** - check for: `openai`, `@anthropic-ai/sdk`, `ai` (Vercel AI SDK), `langchain`, `llamaindex`, `@google/generative-ai`, `cohere-ai`, `replicate`

**Payments** - check deps and code for: stripe, paddle, lemonsqueezy, paypal

**File uploads** - check for: multer, formidable, busboy, express-fileupload, python-multipart, uploadthing

**Infra** - check for: `Dockerfile`, `docker-compose.yml`, `vercel.json`, `netlify.toml`, `fly.toml`, `wrangler.toml`, `railway.json`, `.github/workflows/`

**MCP** - check for MCP config files in the project

Report a one-line summary of what you detected, then move to Phase 2. Example:
> **Detected:** Next.js 14 + Prisma + PostgreSQL, Stripe payments, NextAuth, Docker, GitHub Actions

### Phase 2: Scan

Read `checks/_index.md` in this skill's directory to decide which check files to load based on what was detected in Phase 1.

**Always load** the CORE checks (secrets, injection, auth, config, dependencies, data-exposure).

**Conditionally load** checks only when the matching stack was detected (nextjs, clerk, supabase, firebase, mongodb, ai-llm, payments, uploads, docker, integrations).

For each loaded check file, run the patterns using Grep and Glob tools. Exclude `node_modules`, `.git`, `vendor`, `dist`, `build`, `.next`, `__pycache__`, `venv`, `.venv` from all searches.

Before reporting findings, read `gotchas.md` in this skill's directory to avoid known false positives and common mistakes.

### Phase 3: Report

Use the template in `templates/report.md` for the output structure.

**Scoring criteria (strict):**
- **A**: 0 critical, 0 high, ≤3 medium
- **B**: 0 critical, 1-2 high, any medium
- **C**: 0 critical, 3+ high
- **D**: 1-2 critical findings
- **F**: 3+ critical findings, or any active credential exposure in committed code

**IMPORTANT: Credential redaction rule.** When reporting secret findings, NEVER reproduce the full secret value in the output. Always redact: show only the first 4 characters followed by `****` (e.g., `sk-pr****`, `AKIA****`, `ghp_x****`). In Before/After code blocks for secret findings, use the redacted form. This prevents accidental exfiltration of credentials through the report itself.

### Phase 4: Fix mode

After presenting the report, ask the user:

"Want me to fix these issues? I can go through them one by one."

If the user says yes:

1. Work through findings from highest severity to lowest
2. For each finding, show what you're about to change and apply the fix
3. Skip findings that require external action (e.g., rotating leaked keys, enabling RLS in Supabase dashboard)
4. After all fixes are applied, re-run the checks on the modified files to confirm the issues are resolved
5. Report what was fixed and what still needs manual attention

If the user says no or doesn't respond, end with the report.

### Phase 5: Next steps

If the project uses MCP servers or AI agents, suggest the user look into auditing those configurations separately.

### Examples

**Example 1: "I built this SaaS with Cursor, is it secure?"**
1. Detect: Next.js 14 + Supabase + Stripe + Vercel
2. Load: CORE + nextjs + supabase + payments checks
3. Scan. Focus on: Supabase RLS, Stripe webhook verification, secrets in `.env`
4. Report: Score D - service_role key in client code, no RLS on 3 tables, Stripe webhook unverified
5. Top 3: move service_role to server, enable RLS, add webhook signature check

**Example 2: "Security review before launch"**
1. Detect stack, load relevant checks
2. Scan production config vs dev config - verify debug is off, CORS is scoped, security headers exist
3. Report with quick wins section first for pre-launch fixes

**Example 3: Clean project**
1. Detect and scan all relevant checks
2. Report: Score A - no findings. List what the codebase does well

### Common Issues

**Large codebases:** Prioritize by severity - secrets first (biggest immediate risk), then auth, then payments. Don't try to scan everything in one pass. Skip `node_modules`, `vendor`, build artifacts.

**Many false positives on hardcoded secrets:** Apply context rules strictly (see `checks/secrets.md`). Test files, example configs, and placeholder values are INFO, not CRITICAL. If unsure, include the finding but mark it as "verify manually".

**npm audit / pip audit not available:** Skip the dependency audit command. Report that the tool wasn't available and recommend the user run it manually.

**Project uses a framework not listed:** Apply the general patterns (secrets, injection, auth) even if framework-specific patterns don't match. The core checks work across any stack.
