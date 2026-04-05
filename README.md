# supabase-keep-alive

A GitHub Actions workflow that automatically writes to your Supabase free-tier project twice a week to prevent it from being paused due to inactivity.

> **This repo is a template for demonstration — no workflow is running here.**
> Follow the setup guide below to use it in your own repo.

---

## Background

If you're on Supabase's free tier, you may receive emails like this after 7 days of inactivity:

> *Your project has been paused. To optimize cloud resources, we automatically pause free-tier projects after 7 days of inactivity.*

Manually logging in every few days just to keep a project alive is tedious. This repo automates that with a single GitHub Actions workflow — no extra server or cron job needed.

## How It Works

Simply pinging the Supabase REST API root (`/rest/v1/`) is **not sufficient** — Supabase only counts actual database operations as activity.

This workflow performs a real UPSERT to a dedicated `keepalive` table on a twice-weekly schedule, guaranteeing the project stays active.

---

## Step-by-Step Setup Guide

### Step 1 — Create your own repo and add the workflow

Create a new GitHub repository, then create the file `.github/workflows/supabase-ping.yml` and paste in the contents of [`workflow-template/supabase-ping.yml`](workflow-template/supabase-ping.yml) from this repo.

Alternatively, **fork** this repo — the workflow file is already in place.

---

### Step 2 — Create the keepalive table in Supabase

Go to your Supabase project → **SQL Editor** and run:

```sql
CREATE TABLE IF NOT EXISTS keepalive (
  id integer PRIMARY KEY DEFAULT 1,
  pinged_at timestamptz DEFAULT now(),
  CONSTRAINT single_row CHECK (id = 1)
);

INSERT INTO keepalive (id, pinged_at) VALUES (1, now())
ON CONFLICT (id) DO NOTHING;
```

This creates a single-row table used solely as a ping target. It has no effect on your existing schema.

---

### Step 3 — Get your Supabase credentials

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

### Step 4 — Add GitHub Secrets

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

### Step 5 — Test the workflow manually

Before waiting for the scheduled run, verify that everything works:

1. Go to your repo → **Actions** tab
2. In the left sidebar, click **Supabase Ping**
3. Click **Run workflow** → **Run workflow**
4. Wait about 10–20 seconds, then click into the run
5. Expand the **Ping Supabase Database (UPSERT)** step
6. You should see: `Supabase ping successful (HTTP 200)`

---

### Step 6 — Confirm the ping reached Supabase

To verify the write actually reached the database:

1. Go to [supabase.com](https://supabase.com) and open your project
2. In the left sidebar, click **Table Editor** → `keepalive`
3. Check that the `pinged_at` column shows a timestamp matching when your workflow ran

If the timestamp is updated, the DB write succeeded and your project activity has been registered.

---

## Schedule

```
cron: '0 0 * * 1,4'
```

Runs every **Monday and Thursday** at UTC 00:00. GitHub's scheduler may delay execution by a few minutes — this is normal.

> **Note:** GitHub automatically disables scheduled workflows if a repo has no activity for 60 days. If this happens, manually trigger the workflow once or push any commit to re-enable it.

---

## Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| HTTP 401 / 403 | Wrong or expired Service Role Key | Re-copy the key from Supabase Dashboard → API, then update the GitHub Secret |
| HTTP 404 | `keepalive` table not created or incorrect URL | Run the setup SQL and ensure URL format is `https://<ref>.supabase.co` |
| Workflow not running automatically | Repo inactive for 60+ days | Manually trigger once or push any commit to re-enable |
| Project still paused | Previous approach used a GET ping, not a DB write | Ensure the `keepalive` table exists and the workflow uses POST/UPSERT |
