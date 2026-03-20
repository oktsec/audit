# CONDITIONAL: Third-party Integrations

Only load if OAuth providers or external API calls detected.

## OAuth State Parameter

Without the `state` parameter, an attacker can trick a user into linking the attacker's account (CSRF on OAuth flow).

| Pattern | Issue |
|---------|-------|
| `passport\.authenticate\(` | Check if `state: true` or a custom state parameter is set. If not: HIGH |
| `authorization_url` | Python OAuth: check if `state` is generated and verified on callback |
| `signIn\(` | NextAuth/Auth.js: verify CSRF protection is not disabled |

For any OAuth flow: search for `state` near the authorization URL construction. If the word `state` does not appear near OAuth redirect logic: HIGH.

## Webhook Verification

Any incoming webhook (not just Stripe) must verify the sender's signature. Without it, anyone can POST fake events to your endpoint.

- Search for webhook route handlers (`/webhook`, `/api/webhook`, `/hook`)
- For each webhook endpoint, check if the handler verifies a signature header before processing
- Common verification patterns: `constructEvent`, `verify_signature`, `hmac`, `createHmac`, `webhook_secret`
- If a webhook endpoint processes data without ANY signature check: HIGH

## External API Calls Without Timeout

AI generates HTTP calls to external APIs without timeouts. A slow or dead API hangs your server.

| Call pattern | Language | Timeout to look for |
|-------------|----------|-------------------|
| `fetch\(` | JS/TS | `AbortSignal.timeout` or `signal:` in the options |
| `axios\(` | JS/TS | `timeout:` in the config object |
| `requests\.(get\|post)\(` | Python | `timeout=` parameter |
| `http\.Get\(` | Go | `context.WithTimeout` or `client.Timeout` |

If external API calls exist without any timeout: MEDIUM. One slow third-party API can take down your entire server.
