---
name: 0xinsider-api-access
description: Authenticate against the 0xinsider Developer API and its MCP server, pick between the sandbox and live hosts, and handle rate limits and errors. Use when connecting to 0xinsider, getting an API key, wiring the 0xinsider MCP server into an agent client, debugging a 401, 403 or 429 from api.0xinsider.com, or trying 0xinsider endpoints without an account.
license: MIT
metadata:
  publisher: 0xinsider
  homepage: https://0xinsider.com/developers
  docs: https://docs.0xinsider.com
  openapi: https://0xinsider.com/api/v1/openapi.json
---

# 0xinsider API access

0xinsider is Polymarket analytics & infrastructure for sports and esports markets. This
skill covers getting in: credentials, hosts, transports, and the failure modes
you will actually hit.

## Pick a host first

| Host | Credential | Data |
| --- | --- | --- |
| `https://0xinsider.com/sandbox` | none | Documented examples and deterministic schema samples. Nothing is stored. |
| `https://api.0xinsider.com` | live key, active Pro or Max subscription | Live Polymarket data. |

Start in the sandbox. Every documented operation answers there with no
credential, so you can shape a request and assert on the response schema before
you spend a live quota unit:

```bash
curl -s https://0xinsider.com/sandbox/api/v1/leaderboard
```

Add `?sandbox_status=<code>` to receive one of the error responses that
operation documents. Use it to exercise your retry path:

```bash
curl -si 'https://0xinsider.com/sandbox/api/v1/leaderboard?sandbox_status=429'
```

The sandbox validates documented request bodies and parameters, applies supported
filters to fictional deterministic data, and stores nothing. It does not simulate
streams or file downloads.

## Get a credential with no account

One POST returns a sandbox key and the steps to live access:

```bash
curl -s -X POST https://api.0xinsider.com/api/v1/agents/register \
  -H 'Content-Type: application/json' \
  -d '{"name":"my-agent"}'
```

The key is prefixed `oxi_sk_test_`. It is a sandbox credential: sending it to
`api.0xinsider.com` returns `401 invalid_api_key` with `error.reason` set to
`sandbox_api_key`. That is the expected response, not a bug.

Live keys are prefixed `oxi_sk_live_` and require an active Pro or Max subscription.
Edge Hunter requires Max; Pro access receives `403 forbidden` with
`error.reason: max_plan_required`.

## Authenticate

Two mechanisms, both Bearer:

```bash
curl -s https://api.0xinsider.com/api/v1/leaderboard \
  -H "Authorization: Bearer $OXINSIDER_API_KEY"
```

- **API key** for scripts and server-to-server calls.
- **OAuth 2.1 authorization code with PKCE** for apps and MCP clients. The
  authorization-server metadata is at
  `https://api.0xinsider.com/.well-known/oauth-authorization-server`, and the
  protected-resource metadata follows RFC 9728.

Never hardcode a key. Read it from the environment at call time.

A `401` carries a `WWW-Authenticate` header naming the RFC 9728
`resource_metadata` URL, so a client can discover where to authenticate without
a hardcoded issuer. A `403 insufficient_scope` names the scope an OAuth token
is missing. Read the header rather than guessing.

## Discovery routes that need no credential

- `GET /api/v1` - live route index and capability document
- `GET /api/v1/health`
- `GET /api/v1/coverage` - which data surfaces the API serves
- `GET /api/v1/pick-of-the-day/ledger` - the public pick commitment ledger
- The MCP handshake: `initialize`, `ping`, `tools/list`

Data, export, webhook, and MCP `tools/call` operations require a credential
and an active Pro or Max subscription. Edge Hunter requires Max.
`GET /api/v1/me` and `GET /api/v1/usage` keep answering a valid credential
after paid access lapses, so use them to tell a lapsed plan from a bad key.

## Deprecated paths

These aliases still answer, with `Deprecation`, `Sunset` and a successor
`Link` header, until 2027-02-01. From that date they return `410 Gone` with
`error.code` `endpoint_retired`, and their MCP tools are removed. Write new
calls against the canonical path:

| Deprecated alias | Canonical path | Canonical MCP tool |
| --- | --- | --- |
| `/api/v1/whale-trades` and its sub-paths | `/api/v1/large-trades` | `get_large_trades`, `get_large_trade`, `get_large_trades_history` |
| `/api/v1/insider-radar`, `/api/v1/insider-radar/{id}` | `/api/v1/suspicious-trades` | `get_suspicious_trades`, `get_suspicious_trade` |
| `/api/v1/market/{condition_id}/intel` | `/api/v1/market/{condition_id}/flow` | `get_market_flow` |
| `/api/v1/markets/intel/batch` | `/api/v1/markets/flow/batch` | `batch_get_market_flow` |
| `/api/v1/markets/smart-money-flows` | `/api/v1/markets/sharp-money-flows` | `get_sharp_money_flows` |
| `/api/v1/platforms` | `/api/v1/coverage` | `get_coverage` |

The deprecated MCP aliases are `get_whale_trades`, `get_whale_trade`,
`get_whale_trades_history`, `get_insider_radar`, `get_insider_radar_flag`,
`get_market_intel`, `batch_get_market_intel`, `get_smart_money_flows` and
`get_platforms`. The four legacy sports Edge Hunter routes already return
`410 endpoint_retired`; see "Conditional Edge Hunter reads" below.

## MCP

The remote server speaks Streamable HTTP at
`https://api.0xinsider.com/api/v1/mcp`. Every tool is read-only. Its card,
including transport and tool metadata, is at
`https://0xinsider.com/.well-known/mcp`. `initialize`, `ping` and `tools/list`
need no credential, so `tools/list` is always the current catalog;
`tools/call` takes `Authorization: Bearer` with an API key or an OAuth 2.1
access token, and counts against the same rate limits as REST.

The plugin's `mcp.json` carries no credential: the client signs in through
OAuth on the first `tools/call`. To use an API key, configure the server in a
client that expands `${VAR}` in headers, such as Claude Code's `.mcp.json`:

```json
{
  "mcpServers": {
    "0xinsider": {
      "type": "http",
      "url": "https://api.0xinsider.com/api/v1/mcp",
      "headers": { "Authorization": "Bearer ${OXINSIDER_API_KEY}" }
    }
  }
}
```

`https://0xinsider.com/mcp` answers with the same tools, but the OAuth
protected-resource metadata names `https://api.0xinsider.com/api/v1/mcp` as
the resource, so an OAuth client should connect there.

A local stdio path ships in the CLI:

```bash
npx -y @0xinsider/mcp@latest init
```

Install it permanently with `npm install --global @0xinsider/mcp@latest`, then
run `0xinsider init`, or with `brew install 0xinsider/tap/oxinsider`, which
installs the version its formula pins.

## Conditional Edge Hunter reads

All Edge Hunter views require Max. `GET /api/v1/edge-hunter` defaults to the
quoted board and has no ETag. Select `view=signals` or `view=observations` for
full paginated records and semantic conditional GET. Store the advanced
response's ETag and re-send `If-None-Match`; HTTP 304 has no JSON body. The
TypeScript SDK's `edgeHunter.getConditional()` returns the full selected
collection or `object: "not_modified", data: null`.

Both advanced views use `Cache-Control: private, no-cache`: revalidate
before reuse. All public views read warmed generations only; a missing/stale generation
returns 503 `read_model_warming` with `Retry-After: 5` without computing
holders or calling providers in the request, or fabricating a complete empty result.
A pregame `wider_holder` or `emerging_pile` read for a sport whose holders
have not been read in the last 30 minutes returns the same 503 with
`Retry-After: 45` while those reads restart. Honor the header value rather than
a fixed delay. Parameters for another view return 400;
refreshed snapshots expire cursors. Restart at page one after `cursor_expired`
or when migrating from a retired sports URL. Read `polymarket-sharp-money`
for view-specific queries, source clocks and evidence limits.

## Configurable Edge Hunter selections

Every view accepts `kind=sports|esports` (omit for both), six repeated-string
lists `sports`, `leagues`, `market_types`, `condition_ids`, `event_slugs` and
`series_slugs`, plus legacy `category`. Repeat bare query names or use CSV
inside a value; bracket names are invalid. Lists contain 1..100 supplied values
before normalization/deduplication. OR within an axis, AND across axes.

Send `X-Query-Validation: strict` on selected REST reads and require top-level
`selection` on a selected 200 response. This prevents an older instance from
silently ignoring new selectors. SDK 0.22.0 and MCP 3.2.0 enforce these rules;
empty SDK arrays fail before HTTP. Blank category remains omission behavior.
Advanced cursors and ETags bind the normalized selection; changing it starts
a fresh cursor walk. Unknown provider keys may return an empty selected result.

Each row adds required nullable `market_context`. Its four required nullable
fields are `league`, `series_slug`, `market_type` and `metadata_updated_at`.
Use returned identities for discovery rather than guessing from titles or slug
prefixes. The clock is exactly local `market_canonical.updated_at`, not provider
observation time. Unselected old cache generations may carry context:null;
selected reads require a ready frozen metadata sidecar and return 503 warming
when absent. Missing selected facts exclude rows rather than widening results.

## Rate limits

| Plan | Requests per minute | Batch item units per minute | Included requests per UTC month |
| --- | --- | --- | --- |
| Pro | 100 | 2,500 | 500,000 |
| Max | 200 | 5,000 | 2,000,000 |

Limits are per API-key user. Batch reads accept 25 items per call and cost one
item unit per item, so a batch is never the more expensive choice.

Rate-limit headers are on every response. Read them instead of counting
requests yourself, and honor `Retry-After` on every retryable error: a `429`,
a `503 read_model_warming`, a `503 rate_limit_unavailable`, and a
`404 pick_not_released`. Each also carries `error.retry_at`, an RFC 3339 time,
so you can schedule the retry instead of polling. Check consumption without
spending primary quota (`/api/v1/usage` and `/api/v1/me` share a separate
100-reads-per-minute bucket):

```bash
curl -s https://api.0xinsider.com/api/v1/usage \
  -H "Authorization: Bearer $OXINSIDER_API_KEY"
```

## Errors

Errors are JSON with a typed model: a stable `error.code`, a human `message`,
and an `error.reason` that distinguishes causes sharing one status. Branch on
`error.code`, never on the message string.

Write operations accept `Idempotency-Key`. Send one on every retry of a POST so
a timeout does not become a duplicate.

## Reading the data honestly

An unavailable, partial, or stale provider field marks the edge of the
evidence. Do not render it as a zero and do not describe it as a prediction.
A wallet's P&L is `pnl.in_scope.realized_pnl`; when `pnl.in_scope` is absent
the record is not computed yet, and no other figure substitutes for it. The
wallet-wide `pnl.realized`, `pnl.total` and `pnl.net_total` were retired on
2026-10-27 and are never sent. `polymarket-wallet-grades` has the detail.

## References

- Docs portal: https://docs.0xinsider.com
- OpenAPI 3.1: https://0xinsider.com/api/v1/openapi.json
- Auth guide: https://0xinsider.com/auth.md
- Versioning policy: https://0xinsider.com/api-versioning.md
- Agent instructions: https://0xinsider.com/agents.md
