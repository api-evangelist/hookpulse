---
name: Publish a public status feed
description: >-
  Mint the owner's public status dashboard (JSON + RSS), share it credential-free, and rotate the
  URL if it leaks.
api: openapi/hookpulse-openapi.json
operations: [status_feed_url, get_s_by_token_json, get_s_by_token_rss, status_feed_rotate]
generated: '2026-09-05'
method: generated
---

# Publish a HookPulse status feed

1. **Mint the feed** — `GET /api/status-feed` (`status_feed_url`) with the owner token
   (`X-Guest-Token: hp_…` or `Authorization: Bearer sess_…`). The first call mints the public URL;
   later calls return the same one.
2. **Share it** — the returned URL serves the status of all the owner's monitors with **no header
   at all** (the token in the path is the credential): `GET /s/{token}.json`
   (`get_s_by_token_json`) for dashboards and bots, `GET /s/{token}.rss` (`get_s_by_token_rss`) for
   feed readers and chat integrations (RSS 2.0).
3. **Rotate on leak** — `DELETE /api/status-feed` (`status_feed_rotate`). This is **irreversible
   and immediate**: the previous URL stops working the moment it returns, so update every consumer
   right after rotating.

The feed is free with no quota. Treat the feed URL as a capability token — anyone holding it can
read the status of every monitor the owner has.
