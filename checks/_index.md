# Security Checks Index

Load checks based on what Phase 1 detected. Always load CORE checks. Load CONDITIONAL checks only if the matching stack was detected.

## CORE (always load)

| File | Category | Severity |
|------|----------|----------|
| `checks/secrets.md` | Secrets in code, .gitignore, git history | CRITICAL |
| `checks/injection.md` | SQL injection, XSS, command injection, SSRF, path traversal, deserialization, open redirect | HIGH |
| `checks/auth.md` | Password hashing, randomness, JWT, cookies, rate limiting | HIGH |
| `checks/config.md` | CORS, debug mode, security headers, exposed internals | HIGH |
| `checks/dependencies.md` | Dependency audit, version pinning | MEDIUM |
| `checks/data-exposure.md` | Logging secrets, over-fetching, full objects in responses | MEDIUM |

## CONDITIONAL (load only if detected)

| File | Load when | Severity |
|------|-----------|----------|
| `checks/nextjs.md` | Next.js detected (next in package.json) | HIGH |
| `checks/clerk.md` | Clerk detected (@clerk/* in deps) | HIGH |
| `checks/supabase.md` | Supabase detected (@supabase/* in deps) | CRITICAL |
| `checks/firebase.md` | Firebase detected (firebase/firebase-admin in deps) | CRITICAL |
| `checks/mongodb.md` | MongoDB detected (mongoose/mongodb in deps) | HIGH |
| `checks/ai-llm.md` | AI/LLM deps detected (openai, anthropic, ai, langchain, etc.) | HIGH |
| `checks/payments.md` | Payment deps detected (stripe, paddle, etc.) | HIGH |
| `checks/uploads.md` | Upload handling detected (multer, formidable, busboy, etc.) | HIGH |
| `checks/docker.md` | Dockerfile present | MEDIUM |
| `checks/integrations.md` | OAuth or external API calls detected | HIGH |
