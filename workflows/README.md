# Campaign workflows

These automate the operational edges of the Microsoft Graph email campaign
application, which lives in
[`keithta/Prestige-MCP`](https://github.com/keithta/Prestige-MCP) under
`campaign/`.

| Workflow | Schedule | What it does |
|---|---|---|
| `01-worker-heartbeat.json` | every 5 min | `POST /tick` to the worker. Raises `worker_unreachable` if it does not answer |
| `02-alert-notifier.json` | every 5 min | Emails unnotified alerts, then stamps `notified_at` |
| `03-daily-digest.json` | daily 13:00 UTC | A queue-health summary |

## What these workflows deliberately cannot do

They have **no send authority**. The database enforces this, not convention:
the role they use (`campaign_readonly`) holds SELECT on `alerts` and three
rollup views, and UPDATE on exactly one column, `alerts.notified_at`.

They cannot read contacts or campaigns, cannot claim a job or mark one sent,
cannot change the emergency stop or production mode, and cannot approve, start,
pause or stop a campaign. Ten tests in the application repository
(`campaign/tests/safety/n8n-boundary.test.ts`) assert each of those.

**If you deleted all three workflows, sending would carry on unchanged.** The
worker polls on its own interval; `/tick` only shortens the wait.

That separation is the point. An email is never sent because a workflow fired —
it is sent because the database, at that instant, authorized that specific
email.

## Setup

See [`docs/N8N-SETUP.md`](../docs/N8N-SETUP.md). In short: create a database
credential using the `campaign_readonly` role, a header-auth credential holding
the worker's tick token, set `CAMPAIGN_WORKER_URL` and `CAMPAIGN_ALERT_EMAIL`,
then import the three files.

Before activating anything, confirm the credential is the right role:

```sql
SELECT count(*) FROM campaign.alerts;    -- works
SELECT count(*) FROM campaign.contacts;  -- must fail: permission denied
```

If the second query succeeds, the credential has too much access. Fix that
first.
