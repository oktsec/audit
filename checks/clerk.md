# CONDITIONAL: Clerk Auth

Only load if @clerk/* detected in dependencies.

| Pattern | Issue |
|---------|-------|
| `CLERK_SECRET_KEY` with `NEXT_PUBLIC_` prefix | Secret key exposed to client - CRITICAL |
| `clerkMiddleware\(\)` without `createRouteMatcher` | Middleware doesn't protect any routes by default. Must define protected routes |
| `auth\(\)` | Search Server Components and Actions. Verify `auth()` is called before data access/mutations |
| API routes without `auth()` or `getAuth()` | Unprotected API endpoint |

If Clerk is detected: verify `middleware.ts` exists and uses `createRouteMatcher` to protect routes. Default Clerk middleware without route matching protects nothing.
