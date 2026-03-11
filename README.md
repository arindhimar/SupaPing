# SupaPing 🏓

> **Zero-cost Supabase keep-alive system using GitHub Actions.**

Supabase free-tier projects are paused after **7 days of inactivity**. SupaPing solves this by automatically pinging your Supabase endpoints every 3 days — completely free, no third-party services required.

---

## 📁 Folder Structure

```
SupaPing/
├── services.json               # List of URLs to ping
├── README.md                   # This file
└── .github/
    └── workflows/
        └── keepalive.yml       # GitHub Actions scheduler
```

---

## 🚀 How to Deploy

### 1. Fork / Clone this repository

Push this folder to a **GitHub repository** (public or private — both work for free GitHub Actions minutes).

### 2. Set up your Supabase ping endpoint

You need an HTTP endpoint on your Supabase project that returns a `200 OK` response. Choose one option:

#### Option A – Supabase Edge Function (recommended)

Create a new Edge Function called `ping`:

```typescript
// supabase/functions/ping/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";

serve((_req) => {
  return new Response(
    JSON.stringify({
      status: "alive",
      timestamp: new Date().toISOString(),
    }),
    {
      headers: { "Content-Type": "application/json" },
      status: 200,
    }
  );
});
```

Deploy it with the Supabase CLI:

```bash
supabase functions deploy ping
```

Your endpoint will be:
```
https://<your-project-ref>.supabase.co/functions/v1/ping
```

#### Option B – REST API endpoint

Any existing Supabase REST or PostgREST endpoint that responds with HTTP 2xx will work. For example, a simple table read:

```
https://<your-project-ref>.supabase.co/rest/v1/<your-table>?limit=1
```

> [!NOTE]
> If your endpoint requires authentication, add the `apikey` header to the `curl` command in `keepalive.yml`.

---

### 3. Register your services

Edit `services.json` and replace the example URLs with your own:

```json
[
  "https://your-project-ref.supabase.co/functions/v1/ping",
  "https://your-other-project-ref.supabase.co/functions/v1/ping"
]
```

Add as many URLs as you need — one per line in the JSON array. The GitHub Action will ping every URL in the list.

### 4. Commit and push

```bash
git add .
git commit -m "chore: set up SupaPing keep-alive"
git push
```

GitHub Actions will automatically pick up the workflow and run it on schedule.

---

## ⏰ How the Scheduler Works

The workflow is defined in `.github/workflows/keepalive.yml`.

| Trigger | Detail |
|---|---|
| **Scheduled** | Runs every **3 days** at midnight UTC (`0 0 */3 * *`) |
| **Manual** | Trigger anytime from **Actions → SupaPing → Run workflow** |

On each run, the workflow:
1. Reads every URL from `services.json`
2. Sends an HTTP GET request to each URL (with retry on failure)
3. Logs the timestamp, URL, HTTP status code, and response body
4. **Fails the job** if any ping returns a non-2xx status — so you get an email alert from GitHub

---

## ➕ Adding a New Service

1. Open `services.json`
2. Add your URL to the array:

```json
[
  "https://project1.supabase.co/functions/v1/ping",
  "https://project2.supabase.co/functions/v1/ping",
  "https://project3.supabase.co/functions/v1/ping"   ← add here
]
```

3. Commit and push — no other changes needed.

---

## 📋 Sample Workflow Log

```
==================================================
  SupaPing – Keep-Alive Run
  Timestamp: 2026-03-11 00:00:01 UTC
==================================================

→ Pinging: https://project1.supabase.co/functions/v1/ping
  Time   : 2026-03-11 00:00:02 UTC
  Status : HTTP 200
  Body   : {"status":"alive","timestamp":"2026-03-11T00:00:02.123Z"}
  Result : ✅ OK

→ Pinging: https://project2.supabase.co/functions/v1/ping
  Time   : 2026-03-11 00:00:03 UTC
  Status : HTTP 200
  Body   : {"status":"alive","timestamp":"2026-03-11T00:00:03.456Z"}
  Result : ✅ OK

==================================================
  Summary: 2 passed / 0 failed
==================================================
```

---

## 💡 Tips

- **GitHub Actions free tier** provides 2,000 minutes/month for private repos and unlimited for public repos. Each SupaPing run takes only seconds, so you'll never come close to the limit.
- **Email alerts**: GitHub automatically emails you if a workflow run fails — so you'll know if a ping stopped working.
- **Authenticated endpoints**: If your Supabase endpoint requires an API key, store it as a [GitHub Actions secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets) and pass it as a header:
  ```yaml
  HTTP_CODE=$(curl -s -o /tmp/ping_response.txt -w "%{http_code}" \
    -H "apikey: ${{ secrets.SUPABASE_ANON_KEY }}" \
    "$URL")
  ```

---

## 📜 License

MIT — free to use, modify, and distribute.
