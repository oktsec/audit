# CONDITIONAL: MongoDB

Only load if mongoose or mongodb detected in dependencies.

| Pattern | Issue |
|---------|-------|
| `mongodb\+srv://` with password in connection string | Embedded credentials (use env var) |
| `0\.0\.0\.0/0` in any MongoDB config or comments | Network access open to the entire internet |
| `mongoose\.connect\(` without auth options | Verify auth is configured |
| No `mongoose\.set\('strictQuery'` | May return unexpected fields |
