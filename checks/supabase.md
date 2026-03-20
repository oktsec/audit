# CONDITIONAL: Supabase

Only load if @supabase/supabase-js or supabase detected in dependencies.

## Row Level Security (RLS)

- Search for `supabase.from(` calls. If the app queries tables directly from the client, RLS must be enabled or any authenticated user can read/modify all rows
- Check if a `migrations/` or `supabase/` directory exists with SQL files. Search for `ALTER TABLE.*ENABLE ROW LEVEL SECURITY` and `CREATE POLICY`. If tables exist without RLS policies: CRITICAL
- Search for `service_role` key usage. The service role bypasses RLS - it must NEVER be exposed to the client/frontend

## Key Patterns

| Pattern | Issue |
|---------|-------|
| `service_role` | Supabase service role key (bypasses all RLS - must only be in server-side code, never in client bundle) |
| `SUPABASE_SERVICE_ROLE` | Service role in env (verify: only used server-side) |
| `supabase\.auth\.admin` | Admin auth API (verify: only in server-side code) |
| `NEXT_PUBLIC_SUPABASE_SERVICE_ROLE` | Service role in client bundle - CRITICAL |
