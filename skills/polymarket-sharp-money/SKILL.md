---
name: polymarket-sharp-money
description: Read directional sharp-money and smart-money flow on Polymarket sports and esports markets through the 0xinsider API or MCP server, including market intel, live snapshots, pre-game edge signals, and Insider Radar flags. Use when asked which side sharp money is on, where smart money is moving, what the flow says about a game, or to check anomaly flags on a market.
license: MIT
metadata:
  publisher: 0xinsider
  homepage: https://0xinsider.com/developers
  openapi: https://0xinsider.com/api/v1/openapi.json
---

# Polymarket sharp money

Directional flow across a market, rather than one trade or one wallet. Read
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
| Pre-game signals | `GET /api/v1/sports-edge-signals` | - |
| Observation cohorts | `GET /api/v1/sports-edge-observations` | - |
| Anomaly flags | `GET /api/v1/insider-radar` | `get_insider_radar` |

Markets are keyed by `condition_id`. Resolve a name to one through
`GET /api/v1/markets/search` before you call any of these.

## The sign convention

Market intel reports outcome-aware whale flow. Getting the sign wrong inverts
the conclusion, so it is worth stating plainly:

- `BUY YES` **adds** exposure
- `SELL NO` **adds** exposure
- `BUY NO` **subtracts** exposure
- `SELL YES` **subtracts** exposure

A zero-flow YES tie-break is not conviction. When flow nets to zero, the YES
label is a deterministic tie-break, and reporting it as "sharp money likes YES"
is a fabrication.

## Insider Radar

Insider Radar reads stored scored trades and marks activity for review. Its
live flag floor is 60, and the compatible watch filter stays empty until a
watch policy exists.

A flag does not establish intent, insider knowledge, or a future outcome. It is
a pointer to something worth a human look. The 0-100 suspicion score is a
different quantity from the 0.0-1.0 feed significance score; do not convert or
compare them.

## Sports-edge signals

`GET /api/v1/sports-edge-signals` returns ranked pre-game signals.
`GET /api/v1/sports-edge-observations` returns observation-only cohorts, which
are exactly what the name says: observed, not activated. Category-skill
evidence declares `partial_whale_threshold_fills` or `graded_wallet_fills`
coverage per serving model, and candidate models collect separately until
activation. Read the declared coverage before you quote a cohort.

## Reporting flow without overclaiming

Flow is what graded wallets did. It is not a forecast and not advice. Report
the direction, the size, the window it covers, and the coverage basis. If the
basis is partial, say so in the same sentence as the number.

Keep full numeric precision until the final render.

## Related skills

- `0xinsider-api-access` - credentials, hosts, rate limits
- `polymarket-whale-trades` - the individual fills behind the flow
- `polymarket-wallet-grades` - who is in the cohort
