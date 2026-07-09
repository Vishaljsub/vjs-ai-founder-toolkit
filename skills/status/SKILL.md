# description: Check health of all {{APP_NAME}} services (Docker, backend, DB, queue, storage), template

Copy into `.claude/skills/status/SKILL.md` and fill in the `{{...}}` placeholders. Run this
**before** debugging anything manually — it's the fastest way to rule out "a service just isn't
running" before chasing a real bug.

## Check all Docker services
```bash
docker compose ps
```

## Backend health
```bash
curl -s {{BACKEND_LOCAL_URL}}/health | python -m json.tool
```

## Check backend logs for errors (last 30 lines)
```bash
docker compose logs --tail=30 {{BACKEND_SERVICE}}
```

## Check if DB is reachable (from backend container)
```bash
docker compose exec {{BACKEND_SERVICE}} python -c "{{DB_PING_SNIPPET}}"
```

## Check background worker
```bash
docker compose logs --tail=20 {{WORKER_SERVICE}}
```

## Check all at once (quick summary)
```bash
docker compose ps --format "table {{PS_FORMAT}}"
```

## What to look for
- All services should show `Up` or `running`.
- Backend health endpoint returns a 200 with a healthy status body.
- If the worker is stuck: `docker compose restart {{WORKER_SERVICE}}`.
- If a reverse proxy is stale after a rebuild: restart it — it typically caches upstream IPs at startup.
- If the DB shows `Exit`: `docker compose up -d {{DB_SERVICE}}` then restart backend/worker.

---
*Template — replace every `{{...}}` placeholder with your project's actual values.*
