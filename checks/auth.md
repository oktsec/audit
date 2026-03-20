# HIGH: Authentication and Session Security

## Weak Password Hashing

| Pattern | Issue | Fix |
|---------|-------|-----|
| `createHash\(['"]md5['"]\)` | MD5 is crackable in seconds | Use bcrypt or argon2 |
| `md5\(.*password` | MD5 on password input | Use bcrypt or argon2 |
| `createHash\(['"]sha(1\|256\|512)['"]\)` | SHA is fast, not a password hash (verify: used near password/user context) | Use bcrypt or argon2 |
| `hashlib\.(md5\|sha1\|sha256)\(.*password` | Python: not a password hash | Use bcrypt or argon2 |
| `MessageDigest.*(MD5\|SHA)` | Java: not a password hash (verify: used near password context) | Use BCrypt |

Look for the CORRECT patterns too (bcrypt, argon2, scrypt, pbkdf2). If none found and the app has auth: HIGH.

## Insecure Randomness

| Pattern | Language | Issue |
|---------|----------|-------|
| `Math\.random\(\)` near token, session, id, key, secret, nonce | JS/TS | Predictable output. Use `crypto.randomUUID()` or `crypto.getRandomValues()` |
| `random\.(random\|randint\|choice)\(` near token, secret, password, key | Python | Predictable. Use `secrets.token_hex()` or `secrets.token_urlsafe()` |

## JWT Issues

| Pattern | Issue |
|---------|-------|
| `jwt\.sign\(` | Search for all sign calls. If none include `expiresIn` or `exp` in payload: token never expires |
| `algorithm.*['"]none['"]` | Algorithm none attack - accepts unsigned tokens |
| `algorithms.*\[.*['"]none['"]` | Algorithm none in allowed list |
| `verify.*false` | Signature verification disabled (confirm: in JWT/auth context) |

If `jsonwebtoken` is in deps, also grep for `expiresIn`. If zero matches in the entire codebase: HIGH (tokens never expire).

## Cookie Security

Search for cookie-setting code. Check for:

| Flag | Required | Risk if missing |
|------|----------|----------------|
| `httpOnly: true` | Yes for auth cookies | JavaScript can steal session cookies via XSS |
| `secure: true` | Yes in production | Cookies sent over HTTP in cleartext |
| `sameSite: 'strict'` or `'lax'` | Yes | CSRF attacks |

## Rate Limiting

Check if any rate limiting exists on auth endpoints:
- Node.js: search for `express-rate-limit`, `rate-limit`, `@fastify/rate-limit` in `package.json`
- Python: search for `slowapi`, `django-ratelimit`, `flask-limiter` in deps
- Go: search for rate limit middleware in router setup

If no rate limiting AND auth endpoints exist: HIGH. A login endpoint without rate limiting can be brute-forced.
