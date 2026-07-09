<!--
  Paste this block into your project's CLAUDE.md after you've installed the toolkit
  (see README.md's Quickstart). Fill in the {{...}} placeholders first.
-->

## Change-management loop (read first every session)
The project's history and queue live in four repo files. **Start** each session by reading
[`BACKLOG.md`](BACKLOG.md) (what's next) and the latest [`CHANGELOG.md`](CHANGELOG.md) entry (what
just happened). **End** each session by running the **`wrap`** skill, which updates these files
from what you did:

| File | Holds | When updated |
|---|---|---|
| [`BACKLOG.md`](BACKLOG.md) | Pending queue **only** (prioritised) | Remove finished items, add discovered TODOs |
| [`CHANGELOG.md`](CHANGELOG.md) | Shipped features + **fixes with root cause** | Every session that builds or fixes something |
| [`DEPLOYMENTS.md`](DEPLOYMENTS.md) | What's running where, since when | Only when something is deployed/restarted |
| [`docs/decisions.md`](docs/decisions.md) | Non-obvious architectural/process decisions (the *why*) | Only when such a decision is made |

`Fixed` entries must carry a one-line **root cause** — over time CHANGELOG becomes a searchable
debugging knowledge base. The `wrap` skill is bookkeeping only — it never pushes to production.

## Working style — run commands yourself
Run terminal commands directly via your tools — do not tell the user to run them themselves. You
have a working shell; use it. Only defer to the user for steps that are genuinely interactive and
can't run non-interactively (e.g. a browser OAuth flow like `railway login`, or entering a 2FA
code). After such a one-time step, take over and run the rest yourself.

## Deploy triggers
- **Railway:** `git push origin {{MAIN_BRANCH}}` auto-deploys every service (GitHub-connected
  `{{GITHUB_ORG}}/{{GITHUB_REPO}}@{{MAIN_BRANCH}}`). No CLI deploy step needed — env-var changes
  also auto-redeploy. Verify with `railway deployment list -s {{BACKEND_SERVICE_NAME}}`.
- **GCP Cloud Run:** `git push origin {{MAIN_BRANCH}}` auto-deploys via
  `.github/workflows/gcp-deploy.yml` (Workload Identity Federation, no keys). Full playbook in the
  `deploy-gcp-cloudrun` skill.

Full playbooks with every known gotcha live in the `deploy` and `deploy-gcp-cloudrun` skills —
read those before debugging a failed deploy from scratch.

## Day-to-day skills — use these before doing it by hand
| Skill | When |
|---|---|
| `status` | App not responding? Check this **first** — rules out "a service isn't running" before you debug a real bug |
| `debug` | Hitting a known-shaped error (failed to fetch, streaming error, migration error)? Check here before manual debugging |
| `rebuild` | Just changed code? Targeted rebuild/restart, not a full `docker compose down && up` |
| `logs` | Need to see what a service is doing, local or deployed |
| `backend` / `frontend` | Start or restart a single service |
| `migrate` | Create/apply a DB migration |
| `test-api` | Smoke test the API instead of hand-writing curl commands |
| `wrap` | End of session — update CHANGELOG/BACKLOG/DEPLOYMENTS/decisions.md |

Every non-obvious fix should land a one-line root cause in **both** `debug`'s skill file and
`CHANGELOG.md` — they're meant to reinforce each other into a searchable incident history.
