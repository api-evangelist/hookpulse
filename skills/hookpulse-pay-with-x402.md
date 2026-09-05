---
name: Pay for actions with x402
description: >-
  Handle HookPulse's 402 paywall as an agent — read the price sheet, pay per action with x402
  (USDC on Base), or prepay credit; or take the free 90-day trial instead.
api: openapi/hookpulse-openapi.json
operations: [billing, create_endpoint, post_api_credito, get_api_credito, post_api_auth_start, post_api_auth_verify]
generated: '2026-09-05'
method: generated
---

# Pay HookPulse with x402

1. **Read the price sheet** — `GET /api/billing` (`billing`, no auth). It returns the free
   allowance (10 endpoints, 90s minimum interval, 1 e-mail alert), the unit prices in force
   (extra endpoint $0.10, fast interval $0.05, extra e-mail alert $0.10, agent contact $0.10 — all
   USDC via x402 on Base mainnet), and the account's trial state.
2. **Hit the paywall** — a paid action (e.g. `create_endpoint` past the 10 free, or an interval
   below 90s) returns **HTTP 402** whose body carries `accepts[]` (x402). Settle the payment and
   repeat the **same call** with the `X-PAYMENT` header.
3. **Or prepay credit** — `POST /api/credito` (`post_api_credito`) pays once via x402 and returns a
   `cred_…` bearer token that debits per action on later calls (`Authorization: Bearer cred_…` or
   `X-Credito`). Check balance and statement with `GET /api/credito` (`get_api_credito`) — it never
   returns the token itself. No refund path is documented: only top up what you will use.
4. **Or skip paying entirely** — the trial: `POST /api/auth/start` with `{"email":"…"}`
   (`post_api_auth_start`), receive a 6-digit code, `POST /api/auth/verify`
   (`post_api_auth_verify`). A confirmed e-mail grants **90 days without the usage paywall**
   (extra endpoints and fast intervals free; e-mail alerts beyond the first stay $0.10).

Auth endpoints throttle with **429 + Retry-After**; back off rather than retrying tight loops.
