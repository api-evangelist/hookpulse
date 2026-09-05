---
name: Monitor a cron job with a dead-man switch
description: >-
  Create a HookPulse monitor, wire the cron/webhook to its ingest URL, and read its state — fully
  anonymously, no signup required.
api: openapi/hookpulse-openapi.json
operations: [create_guest, create_endpoint, ping_ingest, get_endpoint, list_events, delete_endpoint]
generated: '2026-09-05'
method: generated
---

# Monitor a cron job with HookPulse

Base URL: `https://hookpulse.net`. All requests/responses are JSON; errors are `{ error, detail? }`.

1. **Mint an owner token** — `POST /api/guest` (`create_guest`, no auth). Save the returned `hp_…`
   token; it is the anonymous owner of your monitors. Send it on later calls as
   `X-Guest-Token: hp_…` or `Authorization: Bearer hp_…`.
2. **Create the monitor** — `POST /api/endpoints` (`create_endpoint`) with
   `{"name":"nightly-backup","interval_sec":900,"alert_to":"you@example.com","alert_url":"https://hooks.slack.com/…"}`.
   `alert_url` must be HTTPS (no localhost). The free tier allows 10 endpoints and a 90s minimum
   interval; past that the call returns **402 with `accepts[]` (x402)** — pay and repeat with
   `X-PAYMENT`, or stay inside the free allowance.
3. **Wire the ping** — have the job hit the ingest URL on schedule:
   `curl -fsS "https://hookpulse.net/in/YOUR_ID"` (`ping_ingest`; `POST /in/:id` also works for
   webhook-only sources). If pings stop past `interval_sec`, HookPulse records a miss and alerts
   (e-mail and/or a POST of a `hookpulse.miss` JSON to `alert_url`, at most once per 24h).
4. **Read state** — `GET /api/endpoints/{id}` (`get_endpoint`) for one monitor,
   `GET /api/endpoints/{id}/events` (`list_events`) for the latest pings. Both also accept the
   monitor's own read-only token (`?token=` / `X-Hook-Token`) so you can share state without the
   owner credential.
5. **Tear down** — `DELETE /api/endpoints/{id}` (`delete_endpoint`). The monitor stops taking pings
   and alerting. No restore is documented, so treat deletion as final.

There is **no idempotency key** on this API — do not blind-retry `create_endpoint`; on a timeout,
list first (`GET /api/endpoints`) and check whether the monitor already exists.
