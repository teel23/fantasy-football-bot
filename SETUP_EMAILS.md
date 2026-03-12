# FF Bot — Email Alert Setup Guide

This guide walks through adding the environment variables needed to enable
injury alert emails and lineup reminder emails.

---

## 1. Railway — Add Env Vars to `ff-bot` Service

Go to **Railway dashboard → ff-bot service → Variables** and add:

| Variable | Value |
|---|---|
| `RESEND_API_KEY` | Get from [resend.com](https://resend.com) → API Keys |
| `ALERT_FROM_EMAIL` | `FF Bot <alerts@yourdomain.com>` — or use `onboarding@resend.dev` for testing (only delivers to your Resend-registered email) |
| `FRONTEND_URL` | `https://ff-bot-web.vercel.app` |

> **Note:** Until you verify a custom domain in Resend, use `onboarding@resend.dev` as the
> from address. It will only deliver to the email address registered with your Resend account.

---

## 2. GitHub — Add Repository Secrets

Go to **github.com/teel23/fantasy-football-bot → Settings → Secrets and variables → Actions → New repository secret** and add each of these:

| Secret Name | Value |
|---|---|
| `SUPABASE_URL` | `https://keksjsxvcymsjhzxubxw.supabase.co` |
| `SUPABASE_SERVICE_KEY` | Supabase dashboard → Settings → API → **service_role** key (not anon key) |
| `RESEND_API_KEY` | Same as Railway |
| `ALERT_FROM_EMAIL` | Same as Railway |
| `RAILWAY_URL` | `https://web-production-dddf8.up.railway.app` |
| `ANTHROPIC_API_KEY` | Your Anthropic API key (needed for lineup reminder AI) |

---

## 3. Test — Manually Trigger an Alert

After adding the Railway env vars, test the injury alert endpoint:

```bash
curl -X POST https://web-production-dddf8.up.railway.app/email-alerts/send-injury-alerts?league_id=1328784373125771264 \
  -H "Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN"
```

Or test a single email (sends mock injury data to your own email):

```bash
curl https://web-production-dddf8.up.railway.app/email-alerts/test \
  -H "Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN"
```

---

## 4. Scheduled Workflows

Once secrets are added, these GitHub Actions run automatically:

| Workflow | Schedule | What it does |
|---|---|---|
| `daily_alerts.yml` | 8am CT daily | Refreshes injury data → emails any user with Out/Doubtful starters |
| `lineup_reminder.yml` | Wed 9pm CT | Sends lineup reminder before Thursday kickoff |

You can also trigger either workflow manually from the **Actions** tab in GitHub.

---

## 5. Email Deduplication

The system tracks sent alerts in the `email_alert_log` Supabase table.
The same injury alert will not be sent more than once per day per player per user.
