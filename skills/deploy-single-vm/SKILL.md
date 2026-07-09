# description: Deploy {{APP_NAME}} to a single VM (SSH + Docker Compose) — manual, no CI/CD, template

For a test/staging box that isn't wired into CI/CD — a manual "tar it up, scp it over, rebuild in
place" runbook. Copy into `.claude/skills/deploy-single-vm/SKILL.md` and fill in the `{{...}}`
placeholders. Prefer the `deploy-railway-gcp` / `deploy-gcp-cloudrun` skills for anything that
should auto-deploy on push — reach for this pattern only when you deliberately want a manually
managed box (e.g. a legacy environment you're keeping alive but not actively syncing).

## Connection facts (fill in once)
| | |
|---|---|
| URL | `{{VM_URL}}` |
| Project/host | `{{CLOUD_PROJECT}}` · zone/region `{{ZONE}}` · instance `{{VM_NAME}}` |
| SSH | `{{SSH_COMMAND_PREFIX}}` (e.g. `gcloud compute ssh {{VM_NAME}} --project {{CLOUD_PROJECT}} --zone {{ZONE}}`) |
| Backend routes | proxied under `{{API_PATH_PREFIX}}` |

## Fast deploy (copy-paste, from repo root)
```bash
P="{{SSH_FLAGS}}"   # e.g. --project X --zone Y

# 1) Package the local tree
tar czf /tmp/{{APP_NAME}}-deploy.tgz --exclude node_modules --exclude .next --exclude .git \
  --exclude '*.tgz' --exclude '.env' --exclude venv \
  --exclude __pycache__ --exclude '.pytest_cache' --exclude '*.pyc' backend frontend

# 2) Copy — IMPORTANT: use a home-RELATIVE filename on the remote side, not `host:~` literally
#    (some SCP clients on Windows don't expand `~` and will save a file literally named "~")
{{SCP_COMMAND}} /tmp/{{APP_NAME}}-deploy.tgz {{VM_NAME}}:{{APP_NAME}}-deploy.tgz $P

# 3) Extract + build + recreate, DETACHED (a build can easily outlive your SSH session window)
{{SSH_COMMAND_PREFIX}} $P --command '
  set -e
  rm -rf {{APP_NAME}} && mkdir {{APP_NAME}} && tar xzf {{APP_NAME}}-deploy.tgz -C {{APP_NAME}}
  # preserve whatever env file already lives on the VM — do NOT overwrite it from local
  cd {{APP_NAME}}
  setsid bash -c "sudo docker compose --env-file {{ENV_FILE_PATH}} \
    -f {{COMPOSE_FILE_PATH}} up -d --build > ~/deploy.log 2>&1" \
    </dev/null >/dev/null 2>&1 & disown'

# 4) Poll until the new code answers (pick a route only the new build has, so you know it rolled)
until [ "$(curl -s -o /dev/null -w '%{http_code}' {{VM_URL}}/{{CANARY_ROUTE}})" = 200 ]; do
  echo "waiting for new code... $(date +%H:%M:%S)"; sleep 15
done
echo "LIVE"
```

## Gotchas (generalized from real incidents — check these first)
- **SCP `~` expansion bug** → always use a filename relative to the remote home dir
  (`host:filename`), never `host:~/filename` — some clients treat `~` as a literal character.
- **`no space left on device` during a build** → large ML/local-model dependencies (torch,
  transformers, etc.) bloat images fast and a build needs headroom for 2 image generations plus
  cache. Fix: prune the builder cache on the VM (`docker builder prune -af`), then finish with
  `--no-build` to just recreate containers from the already-built image if one exists. If this is
  a recurring problem, the permanent fix is usually to move a heavy dependency out to a hosted API
  instead of running it in-process (see the `deploy-gcp-cloudrun` skill's design notes).
- **Build outlives the SSH session** → always run it detached (`setsid ... & disown`) to a
  logfile and poll the endpoint or tail the logfile — don't hold the SSH session open across a
  multi-minute build if your tooling has a session time cap.
- **Migrations** — confirm whether they run automatically on backend startup; if so, named
  volumes for the DB persist across a rebuild so data is safe, but double-check before assuming.

## First admin / seed user (only if the DB is empty)
```bash
{{SSH_COMMAND_PREFIX}} $P --command \
 'sudo docker exec {{DB_CONTAINER_NAME}} psql -U {{DB_USER}} -d {{DB_NAME}} -t -c "SELECT count(*) FROM users;"'
# if 0, register the first admin through the app's own signup/register endpoint
```

## Smoke test
```bash
B={{VM_URL}}
curl -s -o /dev/null -w "health=%{http_code}\n" $B/{{API_PATH_PREFIX}}/health
```

## Day-2 operations
```bash
{{SSH_COMMAND_PREFIX}} $P --command 'cd {{APP_NAME}} && sudo docker compose -f {{COMPOSE_FILE_PATH}} logs -f backend'
{{SSH_COMMAND_PREFIX}} $P --command 'sudo docker compose -f {{COMPOSE_FILE_PATH}} ps'
{{STOP_INSTANCE_COMMAND}}   # stop billing/compute while keeping the disk, if your platform supports it
```

## When to graduate off a single VM
A single VM you deploy to by hand is fine for a short-lived test box, but it doesn't scale and
every deploy is manual toil. The natural upgrade path is a serverless container platform with
push-to-deploy CI/CD (see `deploy-railway-gcp` / `deploy-gcp-cloudrun`) — no SSH, no tar, no disk
pressure, and every `git push` ships itself.

---
*Template — replace every `{{...}}` placeholder with your project's actual values.*
