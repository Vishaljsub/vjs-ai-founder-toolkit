# Deployments — {{APP_NAME}}

What's running where, and since when. Newest first. See the `deploy` skill for the how-to and gotchas.

## Environments
| Env | URL | Deploy trigger | Notes |
|---|---|---|---|
| **Production** | {{PROD_URL}} | `git push origin {{MAIN_BRANCH}}` auto-deploys | |
| **Staging/test** | {{STAGING_URL}} | {{TRIGGER}} | |
| **Local** | http://localhost | manual | Dev only |

---

## Log

| Date | Env | Commit / ref | What | Verified? |
|---|---|---|---|---|
| YYYY-MM-DD | Production | `abcdef1` | one-line summary of what shipped | ✅ / — |
