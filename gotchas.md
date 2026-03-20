# Gotchas

Known failure modes. Read this before reporting findings to avoid common mistakes.

## False Positives

- **`.env.example` and `.env.sample` are NOT leaks.** They contain placeholder values. Only flag `.env` files with real credentials that are not gitignored.
- **`sk_test_` and `pk_test_` are Stripe TEST keys**, not production. Severity: INFO at most.
- **Files in `node_modules/`, `vendor/`, `.git/`, `dist/`, `build/`** are not project code. Never scan or report findings from these directories.
- **SQL patterns in migration files** (`*.sql`, `migrations/`, `alembic/`, `prisma/migrations/`) are expected. Downgrade SQL injection findings in migration files to INFO.
- **`dangerouslySetInnerHTML` with DOMPurify** is safe. Before reporting, check if the input is sanitized with `DOMPurify.sanitize()`, `sanitize-html`, or similar. If sanitized: INFO.
- **`eval()` in build config files** (webpack.config.*, vite.config.*, rollup.config.*, esbuild.*) is normal build tooling, not a vulnerability.
- **`innerHTML` in test files** is not a production risk. Downgrade to INFO.
- **`0.0.0.0` binding inside Docker** is expected (container needs to accept connections from the host network). Only flag if NOT in a Docker context.
- **`cors()` in development-only files** (dev.js, local.js, development config) is fine. Only flag if in production config or the default/shared config.
- **Connection strings in CI/CD files** (.github/workflows/, Jenkinsfile, .gitlab-ci.yml) often use `${{ secrets.* }}` - these are not hardcoded credentials.

## Execution Mistakes

- **Do NOT run `npm audit` if `node_modules/` doesn't exist.** It fails silently or errors. Check first.
- **`git log` for secret scanning can be very slow** in repos with 10K+ commits. Limit to `head -10` or last 100 commits.
- **In monorepos, detect the stack per subdirectory**, not just the root. A monorepo may have a Next.js frontend and a Python backend - both need different checks.
- **Don't grep the entire repo for every pattern.** Use `--type` or `--glob` to scope searches to relevant file types. SQL injection patterns only need to run on `.js`, `.ts`, `.py`, `.go` files, not on `.css` or `.svg`.

## Reporting Mistakes

- **NEVER show full secrets in the report.** First 4 chars + `****`. This is a hard rule, not a suggestion.
- **Don't report the same finding multiple times.** If `Math.random()` appears in 15 test files, report it once with a note "found in 15 files, all in test directories - INFO".
- **The score can inflate artificially** if test files trigger many medium-severity patterns. Apply context rules before counting.
- **Don't report `SELECT *` in seed files, fixtures, or admin/debug scripts** - these are not production query paths.

## Fix Mode Mistakes

- **Never "fix" a leaked credential by just deleting it from code.** The fix is: (1) move to env var, (2) tell user to ROTATE the credential externally. Deleting from code doesn't remove it from git history.
- **Don't add CSP headers without checking** that the app doesn't use inline scripts or styles. A strict CSP can break the entire frontend.
- **Before adding rate limiting to a specific route**, check if there's already a global rate limiter (middleware level). Adding per-route on top of global is redundant.
- **Don't add `httpOnly` to cookies that the frontend JS needs to read** (e.g., theme preference, locale). Only session/auth cookies need httpOnly.
- **When fixing CORS**, don't just replace `*` with `localhost`. Ask what domains need access, or at minimum use the app's own domain from env vars.
