# ☁️ CloudSurf Runner

GitHub Actions workflow that keeps a CloudSurf Codespace alive — waking it automatically every 5 minutes via multiple fallback strategies.

---

## How it works

The workflow runs on a schedule and tries three escalating tricks to wake the Codespace:

1. **`gh codespace start`** — official CLI wake command
2. **SSH ping** — forces the VM runtime to spin up even if the API alone doesn't stick
3. **REST API POST `/start`** — hits the GitHub API directly, bypassing the CLI layer

If none of the tricks immediately return `Available`, it polls every 10 seconds for up to 5 minutes.

---

## Setup

### 1. Fork or clone this repo

### 2. Generate a GitHub Personal Access Token

Go to **GitHub → Avatar → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token**

Check these scopes:
- `repo` — full control of private repositories
- `user` — needed for the REST API call
- `codespace` — full control of codespaces

Copy the token — GitHub only shows it once.

### 3. Add secrets

Go to your repo → **Settings → Secrets and variables → Actions → New repository secret**

| Secret | Value |
|--------|-------|
| `CLOUDSURF_PAT` | Personal access token from step 2 |
| `CLOUDSURF_CODESPACE_NAME` | Your codespace name (run `gh codespace list` to find it) |

> Both values are stored as encrypted secrets and never exposed in logs.

### 4. Push to main

The workflow lives at `.github/workflows/runner.yml` and runs automatically on schedule once pushed.

---

## Schedule

Runs every 5 minutes, continuously.

To change the frequency, edit the `cron` line in `.github/workflows/runner.yml`:

```yaml
- cron: '*/5 * * * *'
```

---

## Manual trigger

Go to **Actions → CloudSurf Runner → Run workflow** to fire it anytime.

---

## Workflow output

Each run logs the codespace state at every step:
📋 Current state: Shutdown
🚀 Trick 1: gh codespace start
State after start: Shutdown
🔌 Trick 2: SSH ping
State after SSH ping: Available
🎉 Done (trick 2)!

If all tricks fail, it polls for X minutes before exiting with a non-zero code (visible as a failed run in Actions).

Here's a section you can drop into both READMEs:

---

## Scheduling via cron-job.org (Recommended over GitHub Actions)

GitHub Actions scheduled workflows are unreliable for frequent intervals — runs can be delayed 30–60+ minutes or dropped entirely. [cron-job.org](https://cron-job.org) is a free, no-card-required alternative that triggers `workflow_dispatch` on your workflow directly via the GitHub API.

### Setup

**1. Create a GitHub Personal Access Token (PAT)**

Go to **GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)**

- Note: `CloudSurf Runner` (or `Video Uploader`)
- Expiration: No expiration (or your preference)
- Scopes:
  - `repo` — required to trigger workflow dispatch
  - `codespace` — required for CloudSurf Runner only

**2. Create a cron-job.org account**

Go to [cron-job.org](https://cron-job.org) → sign up (free, no credit card)

**3. Create the cronjob**

- **Title:** CloudSurf Runner (or Video Uploader)
- **URL:**
  - CloudSurf Runner: `https://api.github.com/repos/<your-username>/CloudSurf-Runner/actions/workflows/main.yml/dispatches`
  - Video Uploader: `https://api.github.com/repos/<your-username>/video-uploader/actions/workflows/uploader.yml/dispatches`
- **Request method:** `POST`
- **Request body:**
  ```json
  {"ref": "main"}
  ```
- **Headers** (click + ADD for each):
  - `Authorization` → `Bearer <your PAT>`
  - `Accept` → `application/vnd.github+json`
  - `Content-Type` → `application/json`
- **Schedule:**
  - CloudSurf Runner: every 30 minutes — `*/30 * * * *`
  - Video Uploader: every 15 minutes — `*/15 * * * *`
- **Timeout:** 30 seconds
- **Notifications:** notify after 3 failures

**4. Remove the schedule trigger from your workflow**

Since cron-job.org handles scheduling, remove the `schedule:` block from your `.yml` file to avoid double-triggering. Keep `workflow_dispatch`:

```yaml
on:
  workflow_dispatch:
```

**5. Test**

Hit **Run now** on the cron-job.org dashboard. A new `workflow_dispatch` run should appear in your GitHub Actions tab within seconds. cron-job.org shows the HTTP response — you want `204 No Content` which means GitHub accepted the trigger successfully.

---

## Related

- [CloudSurf](https://github.com/rrokutaro/CloudSurf) — the main Codespace repo with the browser farm UI
