# CONDITIONAL: Docker

Only load if Dockerfile present.

| Check | Pattern | Issue |
|-------|---------|-------|
| Running as root | Dockerfile without `USER` instruction | Container runs as root |
| Secrets in build | `ARG.*SECRET\|PASSWORD\|KEY\|TOKEN` in Dockerfile | Secrets visible in image layers |
| Latest tag | `FROM.*:latest` | Unpinned base image |
| No .dockerignore | Missing `.dockerignore` file | `.env`, `.git`, `node_modules` may be copied into image |
