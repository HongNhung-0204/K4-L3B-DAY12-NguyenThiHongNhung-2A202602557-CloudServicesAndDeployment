# Checkpoint 5 — Deployment

This checkpoint currently uses the local fallback because no cloud account is
authenticated. The service runs with Docker Compose on this machine; it does
not have a public HTTPS URL yet.
The Cloud Run configuration is prepared for a later deploy after GCP
authentication and an external Redis URL are available.

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
| Platform | Docker Compose local fallback; Cloud Run deployment is pending GCP authentication |
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

## Cloud Deployment Status

`railway.toml`, `render.yaml`, and the Dockerfile remain available as deployment
configuration. This workspace's GCP CLI currently has no active authenticated
account, so no cloud service or public URL has been created. To finish public
deployment on Cloud Run, authenticate the CLI, choose the GCP project, and set
`AGENT_API_KEY` and a reachable managed Redis URL in the service environment.

The local fallback is worth at most 60% of CP5 credit. Set `LOCAL_FALLBACK=true`
in `.env` only while running this fallback; set it to `false` after a public
HTTPS deployment is ready.
