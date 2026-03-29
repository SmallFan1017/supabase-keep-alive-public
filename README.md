# supabase-keep-alive

A GitHub Actions workflow that automatically pings your Supabase free-tier project every 6 days to prevent it from being paused due to inactivity.

## Background

If you're on Supabase's free tier, you may receive emails like this after 7 days of inactivity:

> *Your project is not currently paused, but if it continues not to receive sufficient activity, it will be paused automatically. Once paused, you can unpause it from the dashboard within 90 days — beyond that, the project cannot be restored.*

Manually logging in every few days just to keep a project alive is tedious. This repo automates that with a single GitHub Actions workflow — no extra server or cron job needed.

## How It Works

This workflow sends a `curl` request to the Supabase REST API every 6 days, which counts as project activity and prevents automatic pausing.

---

## Step-by-Step Setup Guide

### Step 1 — Fork or copy this repo

Click **Fork** at the top right of this page to create a copy under your own GitHub account.

If you prefer to start fresh, create a new repo and copy `.github/workflows/supabase-ping.yml` into it.

---

### Step 2 — Get your Supabase credentials

You need two values from your Supabase project.

1. Go to [supabase.com](https://supabase.com) and log in
2. Select your project from the dashboard
3. In the left sidebar, click **Settings** (gear icon at the bottom) → **API**

**Project URL**
- Look for the **Project URL** section at the top of the page
- It looks like: `https://abcdefghijklmn.supabase.co`
- Copy this — this is your `SUPABASE_URL`

**Service Role Key**
- Scroll down to **Project API Keys**
- Find the row labeled `service_role`
- Click **Reveal** to show the full key
- Copy it — this is your `SUPABASE_SERVICE_ROLE_KEY`

> **Why service_role and not anon?**
> The `service_role` key bypasses Row Level Security, ensuring the ping request always succeeds regardless of your database configuration.

> **Is it safe?**
> Yes — the key is stored as a GitHub Secret, which means it's encrypted and never appears in logs or source code.

---

### Step 3 — Add GitHub Secrets

1. Go to your GitHub repo page
2. Click **Settings** (top navigation bar)
3. In the left sidebar, click **Secrets and variables** → **Actions**
4. Click **New repository secret**
5. Add the first secret:
   - **Name:** `SUPABASE_URL`
   - **Value:** paste your Project URL (e.g. `https://abcdefg.supabase.co`)
   - Click **Add secret**
6. Click **New repository secret** again
7. Add the second secret:
   - **Name:** `SUPABASE_SERVICE_ROLE_KEY`
   - **Value:** paste your service_role key
   - Click **Add secret**

You should now see both secrets listed under **Repository secrets**.

> **Note:** `SUPABASE_URL` must not have a trailing `/`

---

### Step 4 — Test the workflow manually

Before waiting 6 days, verify that everything works:

1. Go to your repo → **Actions** tab
2. In the left sidebar, click **Supabase Ping**
3. Click **Run workflow** → **Run workflow**
4. Wait about 10–20 seconds, then click into the run
5. Expand the **Ping Supabase REST API** step
6. You should see: `Supabase ping successful (HTTP 200)`

If you see this message, you're all set. The workflow will now run automatically every 6 days.

---

## Schedule

```
cron: '0 0 */6 * *'
```

Runs every 6 days at UTC 00:00. GitHub's scheduler may delay execution by a few minutes — this is normal.

> **Note:** GitHub automatically disables scheduled workflows if a repo has no activity for 60 days. If this happens, manually trigger the workflow once or push any commit to re-enable it.

---

## Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| HTTP 401 / 403 | Wrong or expired Service Role Key | Re-copy the key from Supabase Dashboard → API, then update the GitHub Secret |
| HTTP 404 | Incorrect URL | Ensure format is `https://<ref>.supabase.co` with no trailing `/` |
| Workflow not running automatically | Repo inactive for 60+ days | Manually trigger once or push any commit to re-enable |
| Empty body warning | No public tables in the database | Harmless — the ping still succeeds |
