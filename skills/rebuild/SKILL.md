# description: Rebuild and restart {{APP_NAME}} services after code changes, template

Copy into `.claude/skills/rebuild/SKILL.md` and fill in the `{{...}}` placeholders. Use this
instead of asking Claude to "refresh and run the application" from scratch — it's the fast,
targeted path.

## Backend changed
```bash
docker compose up -d --build {{BACKEND_SERVICE}}
docker compose restart {{WORKER_SERVICE}}
docker compose logs --tail=20 {{BACKEND_SERVICE}}
```

## Frontend changed
```bash
docker compose up -d --build {{FRONTEND_SERVICE}}
docker restart {{PROXY_CONTAINER_NAME}}
docker compose logs --tail=20 {{FRONTEND_SERVICE}}
```

## Both backend and frontend changed
```bash
docker compose up -d --build {{BACKEND_SERVICE}} {{FRONTEND_SERVICE}}
docker restart {{PROXY_CONTAINER_NAME}}
docker compose logs --tail=30 {{BACKEND_SERVICE}}
```

## Full restart (everything, no rebuild)
```bash
docker compose restart
```

## Full rebuild from scratch (slow — use only for dependency changes)
```bash
docker compose down
docker compose up -d --build
```

## After rebuild: verify the app is up
```bash
curl -s {{BACKEND_LOCAL_URL}}/health
curl -s {{FRONTEND_LOCAL_URL}}/ -o /dev/null -w "%{http_code}"
```

## Key rules (generalized from real incidents)
- If your frontend runs in **production mode** inside Docker, source mounts usually aren't live —
  you must rebuild for any frontend change, not just restart.
- After a frontend rebuild, restart your reverse proxy if you have one — it typically caches
  upstream container IPs at its own startup, not per-request.
- If your backend and background worker share one image, restart the worker after any backend
  rebuild whose change affects task behavior.
- Watch for port conflicts between a Docker service and a locally-run dev server on the same port.

---
*Template — replace every `{{...}}` placeholder with your project's actual values.*
