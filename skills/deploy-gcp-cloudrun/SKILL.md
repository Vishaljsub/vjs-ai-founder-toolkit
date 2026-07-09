# description: GCP Cloud Run + Cloud SQL + GCS serverless stack — architecture, deploy, and CI/CD (push-to-deploy), template

A serverless GCP deployment pattern for a containerized web app. Copy into
`.claude/skills/deploy-gcp-cloudrun/SKILL.md`, fill in the `{{...}}` placeholders after
completing GCP setup (see the toolkit's top-level `SETUP.md`).

**CI/CD: push to `{{MAIN_BRANCH}}` → GitHub Actions auto-deploys** via
`.github/workflows/gcp-deploy.yml` (Workload Identity Federation, no service-account keys) —
so normally you do NOT deploy by hand; just `git push`.

## Architecture (adjust services to your app)
| Piece | Service |
|---|---|
| Frontend | Cloud Run `{{APP_NAME}}-frontend` (public) |
| Backend API | Cloud Run `{{APP_NAME}}-backend` (public) |
| Background worker (optional) | Cloud Run `{{APP_NAME}}-worker` (private, `--min-instances=1` if it needs to stay warm) |
| Database | Cloud SQL (`{{GCP_PROJECT}}:{{REGION}}:{{SQL_INSTANCE}}`) |
| Queue/cache (optional) | Memorystore, or an external managed Redis (e.g. Upstash) if Memorystore's VPC requirement is a blocker |
| Object storage | GCS bucket `{{GCS_BUCKET}}` |
| Images | Artifact Registry `{{REGION}}-docker.pkg.dev/{{GCP_PROJECT}}/{{AR_REPO}}` |
| Secrets | Secret Manager — one secret per credential (`{{APP_NAME}}-secret-key`, `{{APP_NAME}}-db-password`, etc.) |
| Runtime service account | `{{APP_NAME}}-run@{{GCP_PROJECT}}.iam.gserviceaccount.com` (bucket objectAdmin, cloudsql.client, secretAccessor) |
| Deployer service account | `{{APP_NAME}}-deployer@{{GCP_PROJECT}}.iam.gserviceaccount.com` (run.admin, artifactregistry.writer, cloudbuild.builds.editor, storage.admin, iam.serviceAccountUser) |

## Design choices worth keeping (generalized lessons)
- **Keyless auth everywhere.** Use Workload Identity Federation for GitHub Actions → GCP (no
  downloaded JSON keys in CI secrets) and Application Default Credentials for the app's own GCS
  access. Org policies that block service-account key creation (common on managed GCP orgs) make
  this the only option anyway — plan for it from the start rather than retrofitting.
- **Private internal services.** Anything that doesn't need a public URL (a sandboxed code
  executor, an internal worker) should deploy with `--no-allow-unauthenticated` and get called
  with a Google-signed ID token from the caller service, not a shared secret.
- **A dedicated worker entrypoint for Cloud Run.** If your background-job runtime (Celery, BullMQ,
  etc.) doesn't map cleanly onto Cloud Run's "serve HTTP" model, wrap it: serve a trivial health
  endpoint on `$PORT` and run the actual worker loop alongside it in the same process/container.
- **`--args` starting with `-`** (e.g. a `-m module` style entrypoint) needs the `=` form:
  `--args="-m,your.module"` — the space form gets parsed as a gcloud flag, not passed through.

## Gotchas checklist (each line = the fix)
- **Service-account key creation blocked by org policy** → use keyless GCS access + WIF; grant the
  runtime SA `storage.objectAdmin` on the bucket instead of minting HMAC keys.
- **Redis over TLS (`rediss://`)** → many client libraries require an explicit
  `ssl_cert_reqs`/equivalent query param on the URL or they raise at connect time — check your
  broker client's docs before debugging further.
- **Cloud Build permission errors** → grant the **compute default service account**
  (`{{PROJECT_NUMBER}}-compute@developer.gserviceaccount.com`, which runs the build) `storage.admin`
  + `artifactregistry.writer` + `logging.logWriter`; add `.gcloudignore` files per service so local
  `venv`/`node_modules` aren't uploaded to Cloud Build.
- **Migrations racing across multiple instances** → run the backend with a single instance/worker
  during the migration step, or gate migrations behind a separate one-shot Cloud Run Job.
- **`DATABASE_URL` for Cloud SQL** → use the Unix-socket form:
  `postgresql+asyncpg://{{DB_USER}}:<pw>@/{{DB_NAME}}?host=/cloudsql/{{GCP_PROJECT}}:{{REGION}}:{{SQL_INSTANCE}}`
  and deploy with `--add-cloudsql-instances {{GCP_PROJECT}}:{{REGION}}:{{SQL_INSTANCE}}`.
- **Cross-origin cookies** (separate frontend/backend `.run.app` origins) → set
  `COOKIE_SAMESITE=none`, `COOKIE_SECURE=true` on the backend, and bake the resolved backend URL
  into the frontend image as a build substitution so the frontend calls the right origin.

## Manual deploy (rarely needed — CI/CD handles it)
```bash
export PATH="$PATH:{{GCLOUD_SDK_BIN_PATH}}"; cd <repo root>
P="--project {{GCP_PROJECT}} --region {{REGION}}"; AR={{REGION}}-docker.pkg.dev/{{GCP_PROJECT}}/{{AR_REPO}}
gcloud builds submit backend --tag $AR/backend:latest --project {{GCP_PROJECT}} --quiet
gcloud run services update {{APP_NAME}}-backend  $P --image $AR/backend:latest
gcloud run services update {{APP_NAME}}-frontend $P --image $AR/frontend:latest
```

## CI/CD (the normal path)
`.github/workflows/gcp-deploy.yml` (template in `../../github-workflows/gcp-cloudrun.yml`) — on
push to `{{MAIN_BRANCH}}` it authenticates via WIF as the deployer SA, builds each service's image
on Cloud Build tagged `$GITHUB_SHA`, and rolls all Cloud Run services. The WIF provider + deployer
SA identifiers are safe to keep in the workflow file (not secrets) — access is gated by the WIF
trust condition (`assertion.repository=='{{GITHUB_ORG}}/{{GITHUB_REPO}}'`), not by keeping the
identifiers private.

## Verify
```bash
B={{BACKEND_URL}}
curl -s -o /dev/null -w "health=%{http_code}\n" $B/health
gcloud run services logs read {{APP_NAME}}-worker --project {{GCP_PROJECT}} --region {{REGION}} --limit 20
```

---
*Template — replace every `{{...}}` placeholder with your project's actual values, then check this into your repo at `.claude/skills/deploy-gcp-cloudrun/SKILL.md`.*
