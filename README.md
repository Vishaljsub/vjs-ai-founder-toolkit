# Claude Code DevOps + Leadership Toolkit

A portable package of Claude Code skills and agents for two things:

1. **Auto-deployment** — let Claude push code straight to production on Railway and/or GCP Cloud
   Run, with GitHub-connected CI/CD doing the actual build+deploy on every `git push`. Claude's job
   becomes "write the code, commit, push" — not "SSH in and run deploy scripts by hand."
2. **Change management** — a lightweight, file-based history (CHANGELOG / BACKLOG / DEPLOYMENTS /
   decisions log) that Claude reads at the start of every session and updates at the end, so
   context survives across sessions without you re-explaining the project each time.
3. **A virtual leadership team** — advisor agents (Chief of Staff, CGO, CFO, CMO, CTO, Head of
   Sales, Head of People + growth specialists) for a solo/small founder to think through go-to-
   market, pricing, hiring, and roadmap decisions. Advisory only — they never touch code or deploys.

Everything in here is a **template**: `{{PLACEHOLDER}}` tokens mark what you need to fill in for
your own project. Nothing here contains real credentials, URLs, or business-specific content.

## What's in this folder

```
claude-code-toolkit/
├── README.md                          # this file
├── SETUP.md                           # prerequisites + one-time account/CLI setup
├── skills/
│   ├── deploy-railway-gcp/SKILL.md    # Railway + GCP push-to-deploy playbook
│   ├── deploy-gcp-cloudrun/SKILL.md   # GCP Cloud Run architecture + CI/CD detail
│   ├── deploy-single-vm/SKILL.md      # manual single-VM deploy pattern (no CI/CD)
│   ├── wrap/SKILL.md                  # end-of-session change-management bookkeeping
│   ├── backend-service/SKILL.md       # start/manage the backend locally or via Docker
│   ├── frontend-service/SKILL.md      # start/manage the frontend locally or via Docker
│   ├── logs/SKILL.md                  # tail logs for any service, local or deployed
│   ├── status/SKILL.md                # health check across all services before debugging
│   ├── rebuild/SKILL.md               # targeted rebuild/restart after code changes
│   ├── migrate/SKILL.md               # create + apply DB migrations
│   ├── test-api/SKILL.md              # smoke test: health → auth → a few core endpoints
│   └── debug/SKILL.md                 # symptom → root cause → fix, grown from real incidents
├── change-management/
│   ├── CHANGELOG.template.md
│   ├── BACKLOG.template.md
│   ├── DEPLOYMENTS.template.md
│   └── decisions.template.md
├── github-workflows/
│   └── gcp-cloudrun.yml               # GitHub Actions workflow template (WIF-based, no keys)
└── agents/
    ├── README.md
    ├── chief-of-staff.md, chief-growth-officer.md, cfo.md, cmo.md, cto.md,
    │   head-of-sales.md, head-of-people.md
    ├── specialists/                   # positioning, competitive intel, content, demand-gen,
    │                                   #   sales-enablement, launch-pr
    └── skills/                        # repeatable GTM playbooks these agents run
```

The `skills/` folder splits into two groups:
- **Deploy skills** (`deploy-railway-gcp`, `deploy-gcp-cloudrun`, `deploy-single-vm`) — how code
  gets to production.
- **Day-to-day ops + change-management skills** (`wrap`, `backend-service`, `frontend-service`,
  `logs`, `status`, `rebuild`, `migrate`, `test-api`, `debug`) — the habits that keep local dev
  fast and keep CHANGELOG/BACKLOG honest. `status`/`rebuild`/`test-api` exist so Claude reaches for
  a fast, targeted command instead of re-deriving one each time; `debug` and `wrap`'s CHANGELOG
  convention reinforce each other — every non-obvious fix should land a root-cause line in both.

## Quickstart — installing this into a project

1. Read **`SETUP.md`** and install/authenticate the prerequisite tools (git, GitHub CLI, Node,
   Railway CLI, gcloud CLI) and accounts (GitHub, Railway, GCP).
2. Copy what you want into your project's `.claude/` folder and repo root:
   ```bash
   # Deployment skills
   cp -r skills/deploy-railway-gcp   <your-repo>/.claude/skills/deploy
   cp -r skills/deploy-gcp-cloudrun  <your-repo>/.claude/skills/deploy-gcp-cloudrun
   cp -r skills/deploy-single-vm     <your-repo>/.claude/skills/deploy-single-vm   # only if you have one

   # Day-to-day ops + change-management skills
   cp -r skills/wrap             <your-repo>/.claude/skills/wrap
   cp -r skills/backend-service  <your-repo>/.claude/skills/backend
   cp -r skills/frontend-service <your-repo>/.claude/skills/frontend
   cp -r skills/logs             <your-repo>/.claude/skills/logs
   cp -r skills/status           <your-repo>/.claude/skills/status
   cp -r skills/rebuild          <your-repo>/.claude/skills/rebuild
   cp -r skills/migrate          <your-repo>/.claude/skills/migrate
   cp -r skills/test-api         <your-repo>/.claude/skills/test-api
   cp -r skills/debug            <your-repo>/.claude/skills/debug

   # Change-management files (rename off .template.md)
   cp change-management/CHANGELOG.template.md    <your-repo>/CHANGELOG.md
   cp change-management/BACKLOG.template.md      <your-repo>/BACKLOG.md
   cp change-management/DEPLOYMENTS.template.md  <your-repo>/DEPLOYMENTS.md
   mkdir -p <your-repo>/docs
   cp change-management/decisions.template.md    <your-repo>/docs/decisions.md

   # GitHub Actions workflow (GCP Cloud Run path only)
   mkdir -p <your-repo>/.github/workflows
   cp github-workflows/gcp-cloudrun.yml <your-repo>/.github/workflows/gcp-deploy.yml

   # Leadership team agents (optional)
   cp -r agents <your-repo>/.claude/agents-leadership-team
   ```
3. **Find-and-replace every `{{PLACEHOLDER}}`** (product name, founder name, GitHub org/repo, GCP
   project ID, service names, URLs) with your project's real values, across all copied files.
4. Add the snippet in `CLAUDE.md.snippet.md` to your project's `CLAUDE.md` so Claude picks up the
   change-management habit and knows how deploys work automatically.
5. Do the one-time platform setup (Railway project link, GCP service accounts + Workload Identity
   Federation) described in `SETUP.md`.
6. From then on: ask Claude to build something, and when it's ready say **"push this to
   production"** — it commits, pushes, and (if you've set up the workflow) CI/CD takes it from
   there. Run the `wrap` skill (or say "wrap up this session") at the end of a session to keep
   CHANGELOG/BACKLOG/DEPLOYMENTS current.

## Design principles baked into this package

- **Push-to-deploy, not push-a-button.** `git push` is the only deploy action Claude ever needs to
  take. All the actual building/rolling out happens in CI/CD (Railway's GitHub integration, or a
  GitHub Actions workflow using Workload Identity Federation for GCP) — never a manual SSH/gcloud
  session unless something's actually broken.
- **Keyless cloud auth.** The GCP workflow template uses Workload Identity Federation, not
  downloaded service-account JSON keys sitting in GitHub Secrets. Fewer long-lived credentials to
  leak or rotate.
- **File-based memory over chat history.** CHANGELOG/BACKLOG/DEPLOYMENTS/decisions.md are checked
  into the repo, human-readable, and greppable — they survive a fresh Claude Code session (or a
  new teammate) with zero re-explaining.
- **Advisory agents never touch the build.** The leadership-team agents are explicitly told they
  don't edit code, run deploys, or touch build tooling — keeping "thinking about the business" and
  "changing the product" as separate, auditable lanes.
