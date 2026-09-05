# hookpulse
A dead-man's-switch for cron jobs and webhooks. You create a monitor and your cron/Stripe/n8n pings its ingest URL on schedule; if pings stop past a tolerated interval, it records a miss and alerts. Explicitly not website uptime monitoring; its stated primary consumer is AI agents. Offers a public R
