# description: Deploy {{APP_NAME}} to Railway and/or GCP Cloud Run — push-to-deploy playbook, template

Generic push-to-deploy playbook for a containerized app (backend + frontend, optionally a
worker). Copy this into `.claude/skills/deploy/SKILL.md` in your project and fill in the
`{{...}}` placeholders once you've completed the one-time setup in the toolkit's top-level
`SETUP.md`.

## How each environment is deployed (read first)

| Target | Trigger | Mechanism |
|---|---|---|
| **Railway** | `git push origin {{MAIN_BRANCH}}` | Railway is GitHub-connected to `{{GITHUB_ORG}}/{{GITHUB_REPO}}@{{MAIN_BRANCH}}` and **auto-builds & deploys every service on push.** No CLI deploy needed — just commit + push. Env-var changes (`railway variables --set …`) also auto-redeploy that service. |
| **GCP Cloud Run** | `git push origin {{MAIN_BRANCH}}` | `.github/workflows/gcp-deploy.yml` auto-builds + deploys all services via Workload Identity Federation (no long-lived keys). See `deploy-gcp-cloudrun` skill for the full architecture/runbook. |

Both are **push-to-deploy**: once set up, Claude (or you) ships to production by committing and
pushing — no manual CLI deploy step, no copy-pasting build commands.

### Railway — deploy + verify
```bash
git add -A && git commit -m "..." && git push origin {{MAIN_BRANCH}}   # triggers the deploy
railway deployment list -s {{BACKEND_SERVICE_NAME}}     # watch: DEPLOYING → SUCCESS
curl -s {{BACKEND_URL}}/health                          # smoke test once SUCCESS
```

## Pre-deploy checklist
Before pushing for the first time, confirm:
- [ ] All required secrets are set as env vars on each service (API keys, `SECRET_KEY`/`JWT_SECRET`, DB URL) — never commit them
- [ ] `DATABASE_URL` uses the correct async driver prefix for your ORM (e.g. `postgresql+asyncpg://` for SQLAlchemy async)
- [ ] `CORS_ORIGINS` (or equivalent) includes every frontend origin that will call the backend
- [ ] Frontend's `API_URL`/`NEXT_PUBLIC_API_URL` env var is set as a **build** argument (not just a runtime var) if your frontend framework inlines it at build time
- [ ] Object storage / blob-store credentials are set on **every** service that touches storage (backend AND worker, not just backend)

## Services layout (adjust to your app)
```
backend     → API (Dockerfile: backend/Dockerfile)
worker      → background jobs (same Dockerfile, different CMD, if applicable)
frontend    → web app (Dockerfile: frontend/Dockerfile)
postgres    → managed DB
redis       → managed cache/queue (if applicable)
```

## Common deployment gotchas (generalized from real incidents — check these first when a deploy misbehaves)

### Port binding
- PaaS platforms (Railway, Cloud Run) inject a `PORT` env var — never hardcode a port in your start command.
- Container must `EXPOSE` that port and the CMD must bind to `0.0.0.0:$PORT`.
- Framework-specific: Next.js standalone output needs `ENV HOSTNAME=0.0.0.0` in the runner stage. Don't hardcode `ENV PORT=...` in the Dockerfile — let the platform inject it.

### Database
- Use the async driver URL prefix your ORM expects.
- Use the platform's **internal** DB URL for service-to-service traffic; use the **public** URL only for local/admin access.
- Run migrations automatically on startup (e.g. `alembic upgrade head` in your app's lifespan/startup hook) so every deploy is self-migrating — or run them as an explicit CI step before traffic cuts over.

### CORS
- The allow-list must match the deployed frontend origin **exactly** (scheme + host, no trailing slash). Browsers are picky about `http://localhost` vs `http://localhost:80`.

### "Failed to fetch" / frontend can't reach backend after deploy
Root cause 95% of the time: the frontend's API base URL is wrong, missing, or was set as a runtime var when the framework needed it at **build** time.
1. Confirm it's a **build** argument if your framework inlines env vars into the client bundle.
2. Confirm it has the right scheme (`https://`) and no trailing slash.
3. Rebuild (not just restart) the frontend service after changing it.

## Run migrations against a deployed environment
```bash
# Railway
npm install -g @railway/cli
railway login
railway link           # select your project
railway run --service {{BACKEND_SERVICE_NAME}} <your-migration-command>

# Cloud Run — see deploy-gcp-cloudrun skill; migrations typically run on backend startup instead.
```

## Logs
```bash
railway logs --service {{BACKEND_SERVICE_NAME}}
railway logs --service {{WORKER_SERVICE_NAME}}
railway logs --service {{FRONTEND_SERVICE_NAME}}
```

## After deploy: smoke test
```bash
curl -s {{BACKEND_URL}}/health
```

---
*Template — replace every `{{...}}` placeholder with your project's actual values, then check this into your repo at `.claude/skills/deploy/SKILL.md`.*
