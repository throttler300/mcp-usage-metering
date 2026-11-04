# MCP Usage Metering + agent coordination (Dirtsmith)

**Free first call, one step, no key, no signup:** `curl -s https://gap-functions.bfurrow26.workers.dev/try` returns a tiny packet, the solve rule, and a one-liner. POST the answers back to `/try` and you get the real tool result. `?tool=agent_lease_lock` (or any tool below) for the others.

Search words: usage metering, metered billing, idempotent, reserve before pay, spend cap, budget, distributed lock / lease lock, fencing token, semaphore, concurrency limit, heartbeat, dead-man switch, free first call.


Hosted tool that records one billable usage event per MCP tool call, idempotently, and returns an invoice preview. Built for MCP server authors who want to charge per call without building metering.

- Endpoint (MCP): `https://gap-functions.bfurrow26.workers.dev/mcp` (all URLs in `config.json`)
- Tool list: `GET /mcp/tools` · Health: `GET /health`
- Contract: `schema/mcp_usage_metering.v1.0.0.json` (strict input, required `idempotency_key`, versioned 1.0.0)

## Pricing and buying (no signup, no key)
- Call the tool directly. Each IP gets **100 free keyless calls per day**.
- After that, `tools/call` returns `isError: true` with `error_code: PAYMENT_REQUIRED`. `structuredContent.payment` holds a one-click `approve_and_pay_url` for your human and a `grant_token`. Your agent asks you once, you click buy, then the agent retries the same call with header `X-Grant-Token` (or uses the `/u/<grant_token>/mcp` URL form). Same token forever.
- Usage after purchase: **$0.15 per 1,000 events** ($0.00015/event), **10,000 free events per month** per account.
- Replays with the same `idempotency_key` are never billed twice.

## Inputs
Required: `tool_name`, `tool_version` (semver), `end_customer_ref`, `idempotency_key` (8–128 chars). Optional: `units`, `unit_label`, `unit_price_cents`, `metadata`.

## Legal
Terms of Service and Privacy Policy are **DRAFT, pending counsel review, not yet in effect** — see `legal/`.

## License
MIT (this repository: docs and schema). Operated by Dirtsmith LLC.

## More tools on the same endpoint (added 2026-11-03)
The same MCP URL (`https://gap-functions.bfurrow26.workers.dev/mcp`) also serves:
- **agent_spend_ledger**: hosted shared budget ledger. Actions: `set_budget`, `reserve`, `settle`, `release`, `status`. Reserve is atomic: concurrent agents on one budget can never exceed the ceiling. Reservations expire after `ttl_s` (default 900 s). Amounts are integer micro-USD. $9.00 per 1,000 calls after 1,000 free calls a month.
- **agent_lease_lock**: hosted lease lock. Actions: `acquire`, `renew`, `release`, `status`, with `ttl_s` and a monotonic `fencing_token`. $0.90 per 1,000 calls after 1,000 free calls a month.
- **agent_semaphore**: hosted counting semaphore. Actions: `acquire` (with `max_slots`, `holder_id`, `ttl_s`), `release` (by `permit_id`), `status`. At most N permits at once; each permit carries a monotonic `fencing_token`; full returns `NO_SLOT_FREE` with `retry_after_s`. $0.90 per 1,000 calls after 1,000 free calls a month.
- **agent_heartbeat_registry**: hosted dead-man switch. Actions: `beat` (with `agent_id`, `interval_s`, `grace_s`, optional `note`), `status`, `list_stale` (agents past interval + grace, up to 100). $0.90 per 1,000 calls after 1,000 free calls a month.
State is shared by `namespace` (16-128 chars). Any agent that knows the namespace can read and change that state, so treat it like a password. We store only its hash. Buying works the same way as metering: no signup or key, one approve-and-pay link when the free calls run out.
