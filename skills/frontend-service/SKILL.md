# description: Start or manage the {{APP_NAME}} frontend, template

Copy into `.claude/skills/frontend/SKILL.md` and fill in the `{{...}}` placeholders.

## Start via Docker Compose
```bash
docker compose up -d {{FRONTEND_SERVICE}} {{PROXY_SERVICE}}
docker compose logs -f {{FRONTEND_SERVICE}}
```

## Start locally (hot reload)
```bash
cd {{FRONTEND_DIR}}
{{FRONTEND_DEV_COMMAND}}   # e.g. npm run dev
```

## Rebuild frontend container after package changes
```bash
docker compose up -d --build {{FRONTEND_SERVICE}}
```

## Type-check without running
```bash
cd {{FRONTEND_DIR}}
{{TYPECHECK_COMMAND}}   # e.g. npx tsc --noEmit
```

## Key env vars
- `{{API_URL_VAR}}={{PROXIED_LOCAL_URL}}` (through a reverse proxy when running via Docker)
- `{{API_URL_VAR}}={{DIRECT_LOCAL_URL}}` (when running frontend locally against a local backend directly)

## Port conflicts
If both the Docker frontend container AND a local dev server run simultaneously, the frontend port
will conflict. Stop one before starting the other:
```bash
docker compose stop {{FRONTEND_SERVICE}}
```

---
*Template — replace every `{{...}}` placeholder with your project's actual values.*
