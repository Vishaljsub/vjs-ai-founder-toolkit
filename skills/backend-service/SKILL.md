# description: Start or manage the {{APP_NAME}} backend service, template

Copy into `.claude/skills/backend/SKILL.md` and fill in the `{{...}}` placeholders.

## Start via Docker Compose
```bash
docker compose up -d {{BACKEND_SERVICE}} {{WORKER_SERVICE}}
docker compose logs -f {{BACKEND_SERVICE}}
```

## Start locally (hot reload)
```bash
cd {{BACKEND_DIR}}
{{BACKEND_DEV_COMMAND}}   # e.g. uvicorn app.main:app --reload --port 8080
```

## Restart just the backend after code changes
```bash
docker compose up -d --build {{BACKEND_SERVICE}}
docker compose logs -f {{BACKEND_SERVICE}}
```

## Check backend health
```bash
curl {{BACKEND_LOCAL_URL}}/health
```

## Common env vars needed
- `DATABASE_URL` — DB connection string (mind the async driver prefix if your ORM needs one)
- `{{QUEUE_URL_VAR}}` — queue/broker connection (Redis, etc.), if applicable
- `{{OBJECT_STORAGE_VARS}}` — object storage endpoint/keys/bucket, if applicable
- `SECRET_KEY` / `JWT_SECRET` — signing key for auth tokens
- `{{LLM_API_KEY_VARS}}` — any third-party API keys the backend calls

---
*Template — replace every `{{...}}` placeholder with your project's actual values.*
