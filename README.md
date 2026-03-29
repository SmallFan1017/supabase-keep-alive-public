# supabase-keep-alive

A GitHub Actions workflow that automatically pings your Supabase free-tier project every 6 days to prevent it from being paused due to inactivity.

## How It Works

Supabase pauses free projects after a period of inactivity. This workflow runs a `curl` request to the Supabase REST API on a schedule, keeping the project active automatically — no server required.

## Setup

### 1. Add GitHub Secrets

Go to your repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

| Secret | Value | Where to find |
|---|---|---|
| `SUPABASE_URL` | `https://<project-ref>.supabase.co` | Supabase Dashboard → Settings → API → Project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | `eyJhbGciOiJI...` | Supabase Dashboard → Settings → API → service_role key |

> **Note:** `SUPABASE_URL` should not have a trailing `/`

### 2. Verify

Trigger the workflow manually:

1. Go to **Actions** → **Supabase Ping** → **Run workflow**
2. Check the logs for `Supabase ping successful (HTTP 200)`

## Schedule

Runs automatically every 6 days at UTC 00:00 (Taiwan time 08:00).

```
cron: '0 0 */6 * *'
```

## Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| HTTP 401 / 403 | Wrong or expired Service Role Key | Re-copy the key from Supabase Dashboard and update the Secret |
| HTTP 404 | Incorrect URL | Ensure format is `https://<ref>.supabase.co` with no trailing `/` |
| Workflow not running automatically | GitHub disables schedules after 60 days of repo inactivity | Manually trigger once or push any commit |
| Empty body warning | No public tables in the database | Harmless, can be ignored |
