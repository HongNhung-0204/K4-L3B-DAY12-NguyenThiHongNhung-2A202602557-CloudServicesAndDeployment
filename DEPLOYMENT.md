# Checkpoint 5 — Deployment

This checkpoint currently uses the local fallback. The Render Blueprint is
prepared in `render.yaml`, but the public service has not been created yet, so
there is no public HTTPS URL to record.

**Never put API key values in this file.** Store secrets in `.env` locally or in
the cloud platform's secret manager.

## Student and Repository

| Item | Value |
|------|-------|
| Name | Nguyễn Thị Hồng Nhung |
| Mã học viên (Student ID) | 2A202602557 |
| Repository | https://github.com/HongNhung-0204/K4-L3B-DAY12-NguyenThiHongNhung-2A202602557-CloudServicesAndDeployment |

## Service Status

| Item | Value |
|------|-------|
| Local URL | http://localhost:8000 |
| Public URL | Pending Render deployment |
| Platform | Docker Compose local fallback; Render Blueprint prepared, deployment pending |
| Checked | 2026-09-29 |

## Environment

| Variable | Set | Source |
|----------|-----|--------|
| `PORT` | Yes | Compose default is `8000`; the app also reads the platform-provided value |
| `AGENT_API_KEY` | Yes | Local `.env`, passed to the container through Compose; value is not stored here |
| `REDIS_URL` | Yes | Compose uses `redis://redis:6379/0` on the internal network |
| `RATE_LIMIT_PER_MINUTE` | Yes | Compose default `10` |
| `MONTHLY_BUDGET_USD` | Yes | Compose default `10.0` |
| `LOG_LEVEL` | Yes | Compose default `INFO` |
| `LOCAL_FALLBACK` | Yes | `true` in local `.env` |

## Local Verification

Start the stack and check liveness, readiness, and unauthenticated access:

```powershell
docker compose up -d
docker compose ps
curl.exe -i http://localhost:8000/health
curl.exe -i http://localhost:8000/ready
curl.exe -i -X POST http://localhost:8000/ask `
  -H "Content-Type: application/json" `
  -d '{"question":"Hello"}'
```

Observed responses:

```text
/health  -> 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
/ready   -> 200 {"status":"ready","redis":true}
/ask     -> 401 {"detail":"invalid or missing API key"} without X-API-Key
```

For an authenticated `/ask` request, pass `X-API-Key` from local `.env`; do not
paste the key into this document, a screenshot, or a commit.

## Evidence

The CP5 local fallback test expects at least one image in `screenshots/`. The
local health response can be captured as `screenshots/health.png`; capture the
Compose service status separately if submitting the fallback evidence.

## Render Deployment Status

`render.yaml` defines a Docker web service and a private Key Value datastore in
Singapore. The web service gets `REDIS_URL` from the Key Value connection
string, and Render prompts for `AGENT_API_KEY` when the Blueprint is created.
The key stays in Render's environment settings; do not put its value in this
file, chat, or Git.

To create the service, sign in to Render, choose **New > Blueprint**, connect
this repository and its updated `main` branch, then apply the Blueprint. Enter
`AGENT_API_KEY` only in Render's prompt. Wait for both services to finish
deploying, then copy the web service's public HTTPS URL into the Service Status
table above. Check `/health` and `/ready` on that URL. After the deployment is
live, set `LOCAL_FALLBACK=false` in local `.env` before running the CP5 grader.

Render's free web service can sleep after 15 minutes idle; its first request
after that can take about a minute. Free Key Value can lose its in-memory data
when restarted, which is acceptable for this checkpoint's connectivity check.

The local fallback is worth at most 60% of CP5 credit. Set `LOCAL_FALLBACK=true`
in `.env` only while running this fallback; set it to `false` after a public
HTTPS deployment is ready.
