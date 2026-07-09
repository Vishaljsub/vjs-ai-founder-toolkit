# Setup — prerequisites and one-time configuration

Do this once per machine (software) and once per project (accounts/CLI links). After this, Claude
Code can deploy your project by itself on every `git push`.

**Most of this file is not "instructions for you" — it's a checklist for Claude.** Every step
marked 🤖 is a shell command; paste this file's path to Claude Code and say *"work through
SETUP.md — run everything marked 🤖 yourself, tell me when you hit a 🧑 step."* Only the 🧑 steps
are genuinely stuck to a human: they open a browser for OAuth/2FA, or involve creating an account
Claude has no way to click through for you (and shouldn't — it'd be sharing your credentials with
a tool).

Legend: 🧑 you do this (browser/account) · 🤖 Claude runs this for you (just ask)

## 1. Software to install

| Tool | Why | Who does it |
|---|---|---|
| **Git** | version control; every deploy trigger is a push | 🤖 Claude can run the OS installer (`winget install Git.Git`, `brew install git`, `apt install git`) — or 🧑 https://git-scm.com/downloads |
| **GitHub CLI (`gh`)** *(optional)* | lets Claude open PRs / manage the repo from the terminal | 🤖 `winget install GitHub.cli` / `brew install gh` |
| **Node.js + npm** (LTS) | needed to install the Railway CLI | 🤖 `winget install OpenJS.NodeJS.LTS` / `brew install node` |
| **Railway CLI** | link/inspect/deploy Railway projects from the terminal | 🤖 `npm install -g @railway/cli` |
| **Google Cloud SDK (`gcloud`)** | manage GCP projects, Cloud Run, Cloud Build, IAM | 🤖 can run the installer script, but on Windows this one usually goes smoother if you run the official installer yourself: 🧑 https://cloud.google.com/sdk/docs/install |
| **Docker Desktop** *(optional, local dev only)* | build/run containers locally before pushing | 🧑 GUI installer: https://www.docker.com/products/docker-desktop |
| **Claude Code CLI** | this toolkit assumes you're already running it | already done — you're reading this in it |

Tell Claude: *"install git, gh, node, and the Railway CLI, then verify with `--version`."* It can
run every line in this section except Docker Desktop (GUI installer) and, usually, the gcloud SDK
installer (its installer sometimes wants an interactive shell restart) — flag those two back to
you if the automated install misbehaves.

```bash
# Claude runs this after installing, to confirm everything landed
git --version && node --version && npm --version && railway --version && gcloud --version
```

## 2. Accounts you need — 🧑 you create these

Claude cannot sign up for accounts on your behalf (it would mean handing it your email/payment
credentials). Create these yourself, then hand the session back to Claude:

- A **GitHub account**, with the project pushed to a repo you own (or have push access to).
- A **Railway account** (https://railway.app) — free tier is enough to start.
- A **GCP account** (https://console.cloud.google.com) with billing enabled — Cloud Run/Cloud
  Build/Cloud SQL have free tiers but require a billing account attached.

## 3. Railway setup

🧑 **You:** run the login (opens a browser, needs your credentials/2FA):
```bash
railway login
```

🤖 **Claude, from here:**
```bash
railway init              # inside your repo, or:
railway link               # to link to an existing Railway project
railway variables --set "KEY=VALUE" -s <service>   # set each required secret/env var
```

🧑 **You (dashboard, one click, no CLI equivalent):** Settings → Source → connect the GitHub repo
+ branch for each service. This is the one piece that has no CLI/API path — it's what makes
`git push` auto-deploy from then on. Once connected, hand it back to Claude — it can read the
resulting service URLs with `railway status`/`railway domain` and use them to fill in the `deploy`
skill's placeholders and your CORS config.

## 4. GCP setup (only if you're also deploying to Cloud Run)

🧑 **You:** run the login (opens a browser):
```bash
gcloud auth login
```

🤖 **Claude, from here — this entire block is copy-paste shell, hand it to Claude verbatim:**
```bash
gcloud config set project <YOUR_PROJECT_ID>
gcloud services enable run.googleapis.com cloudbuild.googleapis.com \
  artifactregistry.googleapis.com secretmanager.googleapis.com sqladmin.googleapis.com \
  iamcredentials.googleapis.com

# Runtime SA — what your deployed services run as
gcloud iam service-accounts create <APP_NAME>-run --display-name "runtime"

# Deployer SA — what GitHub Actions assumes via WIF to build+deploy
gcloud iam service-accounts create <APP_NAME>-deployer --display-name "CI/CD deployer"
for role in run.admin artifactregistry.writer cloudbuild.builds.editor storage.admin iam.serviceAccountUser; do
  gcloud projects add-iam-policy-binding <YOUR_PROJECT_ID> \
    --member="serviceAccount:<APP_NAME>-deployer@<YOUR_PROJECT_ID>.iam.gserviceaccount.com" \
    --role="roles/$role"
done

# Workload Identity Federation — so GitHub Actions can auth without a downloaded key
gcloud iam workload-identity-pools create github-pool --location=global
gcloud iam workload-identity-pools providers create-oidc github-provider \
  --location=global --workload-identity-pool=github-pool \
  --issuer-uri="https://token.actions.githubusercontent.com" \
  --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository" \
  --attribute-condition="assertion.repository=='<GITHUB_ORG>/<GITHUB_REPO>'"
gcloud iam service-accounts add-iam-policy-binding \
  <APP_NAME>-deployer@<YOUR_PROJECT_ID>.iam.gserviceaccount.com \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/<PROJECT_NUMBER>/locations/global/workloadIdentityPools/github-pool/attribute.repository/<GITHUB_ORG>/<GITHUB_REPO>"

# Claude can read these back out and fill them into the workflow file itself:
gcloud iam workload-identity-pools providers describe github-provider \
  --location=global --workload-identity-pool=github-pool --format="value(name)"
```

🤖 Claude should then **edit `github-workflows/gcp-cloudrun.yml` itself** — filling in
`WIF_PROVIDER` and `DEPLOY_SA` from the command output above — rather than you copying values
around by hand.

## 5. Wire it into your repo — 🤖 all Claude

Tell Claude: *"copy the toolkit's skills, change-management templates, and CLAUDE.md snippet into
this repo, and fill in every `{{PLACEHOLDER}}` with this project's actual values."* It has
everything it needs (repo name, GitHub remote, service names it just created) to do this without
you supplying each value by hand — it only needs to ask you for things it can't infer, like which
branch is "main" if that's ambiguous, or your product name/tagline for the leadership-team agents.

## 6. First deploy — 🤖 all Claude

```bash
git add -A && git commit -m "chore: wire up push-to-deploy"
git push origin <your-main-branch>
```
Claude can watch the rollout itself:
```bash
railway deployment list -s <service>          # poll until SUCCESS
gh run watch                                   # poll the GitHub Actions run, if using GCP
```
Once both are green, every future push deploys automatically — from here on, "ship this" just
means asking Claude to commit and push.

---

**Bottom line:** of the six sections above, only 3 short 🧑 items are actually stuck to you —
create the accounts, and run `railway login` / `gcloud auth login` once each (both are one-line
browser logins). Everything else — installs, IAM, WIF, repo wiring, the first deploy, and every
deploy after — is a shell command Claude Code can run for you if you just ask it to work through
this file.
