# description: Debug common {{APP_NAME}} errors — maps symptoms to root causes and fixes, template

Copy into `.claude/skills/debug/SKILL.md` and fill in the `{{...}}` placeholders. The point of
this skill is to turn each real incident into a permanent, searchable symptom → root cause → fix
entry — pair it with the `wrap` skill's CHANGELOG convention (every `Fixed` entry gets a one-line
root cause) so this file and CHANGELOG.md reinforce each other over time. Start empty-ish and add
a section every time you actually debug something non-obvious; don't try to pre-fill this with
speculative entries.

## "Failed to fetch" on login (local)
**Root cause options (check in order):**
1. Backend not running → `curl {{BACKEND_LOCAL_URL}}/health`
2. CORS mismatch → check the backend's CORS origins include your frontend's local origin
3. Frontend pointing at the wrong backend URL → check `{{API_URL_VAR}}` in the frontend's local env file
4. Reverse proxy not routing → hit a backend route through the proxy directly and check the status code

## "Failed to fetch" in production
1. `{{API_URL_VAR}}` missing or wrong — if your frontend framework inlines env vars at build time, it must be set as a **build** argument, not a runtime one, with the correct `https://` URL
2. CORS origin list missing the deployed frontend URL in backend env vars
3. Backend crashed on startup — check platform logs for a startup exception (missing secret, bad DB URL, etc.)

## Streaming errors (SSE/WebSocket)
1. Check reverse proxy timeout settings — long-lived streaming connections need a generous read/idle timeout, not the platform default
2. Check backend logs for exceptions raised mid-stream
3. Verify the event/message types the backend sends match what the frontend's stream parser expects

## Background job / worker not processing
1. Worker container not running → `docker compose ps`, check its status
2. Broker/queue not configured → worker logs will show a connection error
3. A dependent third-party API key missing/expired
4. Wrong resource name/bucket referenced (a common silent-failure cause — the resource exists, just under a different name than what's configured)

## Migration errors on startup
1. Confirm the migration command's working directory matches wherever your app's startup hook expects it
2. Check current migration state: `... alembic current` (or your migration tool's equivalent)
3. If migrations conflict: review history rather than guessing
4. Never delete an already-applied migration file

## Auth / permission "Failed to fetch" on a specific action
1. Check auth token/cookie is actually being sent (`credentials: "include"` on fetch, or equivalent)
2. Check backend logs for validation errors (missing required fields is a common false "auth" symptom)
3. If multi-tenant: confirm the tenant/org scope is present on every request path involved

## Duplicate responses / double-fired requests
Root cause is almost always a stream or subscription being opened twice. Check:
1. Only one connection/reader should be open per logical request
2. Cleanup on component unmount/re-render (in a frontend framework, a missing `useEffect` cleanup is the classic cause)

## Reverse proxy stale after a service rebuild
```bash
docker restart {{PROXY_CONTAINER_NAME}}
```
Most reverse proxies cache upstream service IPs at their own startup. Any time a backend or
frontend container is recreated, restart the proxy too.

## Port conflict
Stop the Docker version of a service before running its local dev-server equivalent (or vice versa):
```bash
docker compose stop {{FRONTEND_SERVICE}}
```

## Backend won't start: a required secret is missing or still the default
Set a real value for the flagged env var. A startup check that refuses to boot on a default/dev
secret is a feature, not a bug — don't work around it, fix the env var.

## Worker stuck / tasks not processing
```bash
docker compose restart {{WORKER_SERVICE}}
docker compose logs --tail=50 {{WORKER_SERVICE}}
```

---
*Template — replace every `{{...}}` placeholder with your project's actual values, and keep
adding real, project-specific entries as you debug things. This file is most valuable once it
reflects your actual incident history, not the starter list above.*
