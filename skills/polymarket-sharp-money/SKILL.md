---
name: polymarket-sharp-money
description: Read directional sharp-money and smart-money flow on Polymarket sports and esports markets through the 0xinsider API or MCP server, including market intel, live snapshots, the Max-only Edge Hunter board, and Insider Radar flags. Use when asked how graded-wallet flow is distributed across a market's outcomes, where high-grade wallets are concentrating, how to read a market's flow analytics, or to check review flags on a market.
license: MIT
metadata:
  publisher: 0xinsider
  homepage: https://0xinsider.com/developers
  openapi: https://0xinsider.com/api/v1/openapi.json
---

# Polymarket sharp money

Flow analytics across a market, rather than one trade or one wallet: how
the activity of graded wallets is distributed across a market's outcomes. Read
`0xinsider-api-access` first for credentials and hosts.

## Calls

| Need | Call | MCP tool |
| --- | --- | --- |
| Ranked sharp flow | `GET /api/v1/markets/sharp-money-flows` | `get_sharp_money_flows` |
| Ranked smart flow | `GET /api/v1/markets/smart-money-flows` | `get_smart_money_flows` |
| One market | `GET /api/v1/market/{condition_id}/intel` | `get_market_intel` |
| Many markets | `POST /api/v1/markets/intel/batch` | `batch_get_market_intel` |
| Live snapshot | `GET /api/v1/market/{condition_id}/snapshot` | `get_market_snapshot` |
| OHLC candles | `GET /api/v1/market/{condition_id}/candles` | - |
| Edge Hunter board (Max) | `GET /api/v1/edge-hunter` | `get_edge_hunter` |
| Review flags | `GET /api/v1/insider-radar` | `get_insider_radar` |

Market-specific calls use `condition_id`. Resolve a name through
`GET /api/v1/markets/search` before calling them. Edge Hunter returns the
ranked all-sports board without a market identifier.

## The sign convention

Market intel reports outcome-aware large-trade flow. Getting the sign wrong
inverts the conclusion, so it is worth stating plainly:

- `BUY YES` **adds** exposure
- `SELL NO` **adds** exposure
- `BUY NO` **subtracts** exposure
- `SELL YES` **subtracts** exposure

A zero-flow YES tie-break is not conviction. When flow nets to zero, the YES
label is a deterministic tie-break, and reporting it as a lean toward YES is a
fabrication.

## Insider Radar

Insider Radar reads stored scored trades and marks activity for review. Its
live flag floor is 60, and the compatible watch filter stays empty until a
watch policy exists.

A flag does not establish intent, non-public knowledge, or a future outcome.
It is a pointer to something worth a human look. The 0-100 review score is a
different quantity from the 0.0-1.0 feed significance score; do not convert or
compare them.

## Edge Hunter

Use `GET /api/v1/edge-hunter` or MCP `get_edge_hunter` for the same ranked
board shown at https://0xinsider.com/sports/edge. Live access requires Max;
a valid Pro credential receives `403 forbidden` with
`error.reason: max_plan_required`.

`horizon_hours` is the only parameter: it defaults to 48 and clamps to 1–48.
The board uses a B-grade floor across sports and returns at most 30 ranked
rows. `total` counts the requested window before that cap; there is no cursor,
category filter or cohort selector.

The REST envelope is `{object: "edge_hunter", data: <board>, meta: <meta>}`.
Read `data.rows`; the authorized board has `viewer: "max"`, and each row has
`state: "full"`, `free: false` and its signal. The MCP tool projects the board
payload with the response metadata beside it.

Read `as_of`, `cached_age_seconds`, `ranking_source` and `directional_source`
from the board to describe the snapshot's age and provenance. Outcome
`current_price` quotes have their own basis and timestamps; a missing quote
is null and the snapshot clock is not the quote clock. The contract omits
token IDs and category-skill evidence. Responses use
`Cache-Control: private, no-store`; there is no ETag/304 path.

The earlier sports feeds are retired and return `410 endpoint_retired`.
Migrate clients to the board envelope and remove their old filters, cursors
and observation-cohort assumptions. The former observation cohorts have no
replacement in Edge Hunter. Follow the current typed contract at
https://docs.0xinsider.com/api-reference/endpoint/get-edge-hunter.

## Reporting flow without overclaiming

Flow is what graded wallets did. It is analytics, not a forecast and not
advice. Report
the direction, the size, the window it covers, and the coverage basis. If the
basis is partial, say so in the same sentence as the number.

Keep full numeric precision until the final render.

## Related skills

- `0xinsider-api-access` - credentials, hosts, rate limits
- `polymarket-whale-trades` - the individual fills behind the flow
- `polymarket-wallet-grades` - who is in the cohort
