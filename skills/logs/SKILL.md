# description: Check logs for any {{APP_NAME}} service, template

Copy into `.claude/skills/logs/SKILL.md` and fill in the `{{...}}` placeholders.

## All services
```bash
docker compose logs -f
```

## Specific service
```bash
docker compose logs -f {{BACKEND_SERVICE}}
docker compose logs -f {{WORKER_SERVICE}}
docker compose logs -f {{FRONTEND_SERVICE}}
docker compose logs -f {{PROXY_SERVICE}}
docker compose logs -f {{DB_SERVICE}}
```

## Last N lines only
```bash
docker compose logs --tail=100 {{BACKEND_SERVICE}}
```

## Check service status
```bash
docker compose ps
```

## Restart a stuck service
```bash
docker compose restart {{BACKEND_SERVICE}}
docker compose restart {{WORKER_SERVICE}}
```

## Deployed environments
```bash
railway logs --service {{BACKEND_SERVICE}}          # Railway
gcloud run services logs read {{APP_NAME}}-backend --project {{GCP_PROJECT}} --region {{REGION}} --limit 50   # Cloud Run
```

---
*Template — replace every `{{...}}` placeholder with your project's actual values.*
