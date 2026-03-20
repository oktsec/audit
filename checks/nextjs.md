# CONDITIONAL: Next.js + Vercel

Only load if Next.js detected in package.json.

## `NEXT_PUBLIC_` Env Var Exposure

Variables prefixed with `NEXT_PUBLIC_` are bundled into the client JavaScript. AI frequently puts server-side secrets here.

| Pattern | Issue |
|---------|-------|
| `NEXT_PUBLIC_.*SECRET` | Secret exposed to browser |
| `NEXT_PUBLIC_.*KEY` that isn't a publishable key | Verify: Supabase `anon` key is OK, but `service_role` key is not |
| `NEXT_PUBLIC_SUPABASE_SERVICE_ROLE` | Service role key shipped to every browser |
| `NEXT_PUBLIC_STRIPE_SECRET` | Stripe secret key in client bundle |
| `NEXT_PUBLIC_DATABASE` | Database credentials in client |
| `NEXT_PUBLIC_.*TOKEN` | Token exposed to browser (verify: is this a public token?) |

Check `.env*` files and `next.config.*` for any `NEXT_PUBLIC_` variable that holds a secret.

## Server Actions and API Routes Without Auth

| Pattern | Issue |
|---------|-------|
| `"use server"` | Search all Server Action files. Verify each exported function checks auth before mutating data |
| `app/api/.*route\.(ts\|js)` | Check each API route for auth middleware. AI often creates API routes without `getServerSession` or equivalent |
| `export async function (GET\|POST\|PUT\|DELETE)` | Next.js route handler - verify auth check exists |

## Vercel Deployment

- Check `vercel.json` for env vars in plaintext (should use Vercel dashboard, not config file)
- Check if preview deployments have access to production secrets (common misconfiguration)
