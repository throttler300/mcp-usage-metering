# MCP Usage Metering

Hosted tool that records one billable usage event per MCP tool call, idempotently, and returns an invoice preview. Built for MCP server authors who want to charge per call without building metering.

- Endpoint (MCP): `https://gap-functions.bfurrow26.workers.dev/mcp` (all URLs in `config.json`)
- Tool list: `GET /mcp/tools` · Health: `GET /health`
- Contract: `schema/mcp_usage_metering.v1.0.0.json` (strict input, required `idempotency_key`, versioned 1.0.0)

## Pricing
- **$0.15 per 1,000 events** ($0.00015/event)
- **10,000 free events per month**
- Past the free allowance the call returns **HTTP 402 `PAYMENT_REQUIRED`** with a buy link. Your agent asks you once, you click buy, the tool keeps working. Replays with the same `idempotency_key` are never billed twice.

## Inputs
Required: `tool_name`, `tool_version` (semver), `end_customer_ref`, `idempotency_key` (8–128 chars). Optional: `units`, `unit_label`, `unit_price_cents`, `metadata`.

## Legal
Terms of Service and Privacy Policy are **drafts pending review** — see `legal/`.

## License
MIT (this repository: docs and schema). Operated by Dirtsmith LLC.
