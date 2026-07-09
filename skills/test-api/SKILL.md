# description: Smoke test the {{APP_NAME}} API — health, auth, and a few core endpoints, template

Copy into `.claude/skills/test-api/SKILL.md` and fill in the `{{...}}` placeholders. Run this
instead of manually crafting curl commands each time you want to sanity-check the API.

## 1. Health check
```bash
curl -s {{BACKEND_LOCAL_URL}}/health | python -m json.tool
```
Expected: a 200 with a healthy status body.

## 2. Login and get an auth cookie/token
```bash
curl -s -c /tmp/{{APP_NAME}}-cookies.txt \
  {{BACKEND_LOCAL_URL}}/{{LOGIN_PATH}} \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"email": "{{TEST_EMAIL}}", "password": "{{TEST_PASSWORD}}"}' \
  | python -m json.tool
```

## 3. Check current user / session
```bash
curl -s -b /tmp/{{APP_NAME}}-cookies.txt \
  {{BACKEND_LOCAL_URL}}/{{ME_PATH}} \
  | python -m json.tool
```
Expected: a user/session object confirming the auth cookie worked.

## 4–N. A few core read endpoints
List 3–5 endpoints here that exercise the product's core read paths (list agents/items, fetch a
dashboard summary, list resources, etc.) — whatever "is the app basically working" means for your
product. Each one:
```bash
curl -s -b /tmp/{{APP_NAME}}-cookies.txt {{BACKEND_LOCAL_URL}}/{{SOME_CORE_PATH}} | python -m json.tool
```

## Quick all-in-one check (bash script)
```bash
echo "=== Health ===" && curl -s {{BACKEND_LOCAL_URL}}/health
echo ""
echo "=== Login ===" && curl -s -c /tmp/{{APP_NAME}}-cookies.txt {{BACKEND_LOCAL_URL}}/{{LOGIN_PATH}} -X POST -H "Content-Type: application/json" -d '{"email":"YOUR_EMAIL","password":"YOUR_PASS"}' | python -m json.tool
echo ""
echo "=== Me ===" && curl -s -b /tmp/{{APP_NAME}}-cookies.txt {{BACKEND_LOCAL_URL}}/{{ME_PATH}} | python -m json.tool
```

## Against a deployed environment
Replace `{{BACKEND_LOCAL_URL}}` with the deployed backend URL (Railway/Cloud Run). Cookie-based
auth works the same way — use `-c` / `-b` with curl as above.

---
*Template — replace every `{{...}}` placeholder with your project's actual values.*
