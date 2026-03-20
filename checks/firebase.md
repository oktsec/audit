# CONDITIONAL: Firebase

Only load if firebase or firebase-admin detected in dependencies.

## Security Rules

- Check for `firestore.rules` or `database.rules.json`. If missing: CRITICAL (defaults may allow public read/write)
- Search rules files for overly permissive patterns

| Pattern | Issue |
|---------|-------|
| `allow read, write: if true` | Firebase rule allows public access - no auth required |
| `allow read, write: if request.auth != null` | Any logged-in user can read/write all documents (need per-document rules with `resource.data`) |

Rules should use granular per-document checks (`resource.data.userId == request.auth.uid`), not blanket auth checks.
