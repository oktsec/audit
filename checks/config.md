# HIGH: Overly Permissive Configs

## CORS Wildcard

| Pattern | Framework | Issue |
|---------|-----------|-------|
| `cors\(\)` | Express | Defaults to `origin: *` if called with no config object |
| `Access-Control-Allow-Origin.*\*` | Any | Wildcard CORS header |
| `CORS\(app\)` | Flask-CORS | Allows all origins if no `origins=` param |
| `AllowAllOrigins:\s*true` | Go gin-cors | Allows all origins |
| `allowedOrigins.*\*` | Spring | Allows all origins |
| `allowedOriginPatterns.*\*` | Spring | Allows all origin patterns |
| `CORS_ALLOW_ALL_ORIGINS\s*=\s*True` | Django | Allows all origins |

If CORS wildcard found, check if it's only in development config. If it's in production or the default config: HIGH.

## Dangerous Network Binding

| Pattern | Issue |
|---------|-------|
| `listen\(.*['"]0\.0\.0\.0['"]` | Binding to all interfaces (ok inside Docker, risky on bare metal) |
| `host.*['"]0\.0\.0\.0['"]` | Service exposed to network (verify: intentional in container?) |
| `bind.*['"]0\.0\.0\.0['"]` | Accessible from network |

## Debug Mode in Production

| Pattern | Framework | Issue |
|---------|-----------|-------|
| `DEBUG\s*=\s*True` | Django | Full stack traces exposed (skip if in `settings/dev.py` or `local_settings.py`) |
| `app\.debug\s*=\s*True` | Flask | Debug mode with interactive debugger |
| `NODE_ENV.*development` | Node.js | Flag only in Dockerfile or docker-compose (dev mode in container) |
| `enableDevTools.*true` | Various | Dev tools accessible in production |
| `devtool.*true` | Various | Source maps or debug tools enabled |

## Missing Security Headers

Check if security headers are set. Search for `helmet` (Node.js), `django-csp` / `django.middleware.security` (Django), `secure` (Go).

If no security header middleware found and the app serves HTTP responses: MEDIUM.

Key headers to verify:

| Header | Risk if missing |
|--------|----------------|
| `Strict-Transport-Security` | Browser allows HTTP connections (credentials sent in cleartext) |
| `X-Frame-Options` or `frame-ancestors` in CSP | Clickjacking - attacker embeds your app in an iframe |
| `X-Content-Type-Options: nosniff` | Browser MIME-sniffs responses (can execute uploaded files as scripts) |
| `Content-Security-Policy` | No XSS mitigation at browser level |

## Exposed Internals

Manual checks (not pattern-based):

1. Check if the static file server config serves dotfiles (`.git` exposure = full source code leak)
2. Search error handlers for stack trace leaks: look for `err.stack`, `traceback.format_exc()`, `debug.PrintStack()` in response bodies
3. Check if `/api/` routes apply auth middleware. Search for route definitions without `auth`, `protect`, `requireAuth`, `isAuthenticated` middleware
