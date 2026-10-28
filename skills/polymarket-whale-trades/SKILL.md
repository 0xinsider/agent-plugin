---
name: polymarket-whale-trades
description: Read large Polymarket trades on sports and esports markets through the 0xinsider API or MCP server - list the live feed, replay history, inspect one trade's counterparties, and interpret the review score. Use when asked which wallets placed the largest trades on a game, to follow large-trade activity on Polymarket, to build a large-trade alert or feed, or to replay a market's large-trade history before it settled.
license: MIT
metadata:
  publisher: 0xinsider
  homepage: https://0xinsider.com/developers
  openapi: https://0xinsider.com/api/v1/openapi.json
---

# Polymarket large trades

Large trades on Polymarket sports and esports markets, as 0xinsider ingests
them. This is analytics over public activity: the tools describe what was
traded, by which graded wallet, and how unusual it was. Read `0xinsider-api-access` first for credentials and hosts.

## Calls

| Need | Call | MCP tool |
| --- | --- | --- |
| Recent feed | `GET /api/v1/large-trades` | `get_large_trades` |
| One trade | `GET /api/v1/large-trades/{id}` | `get_large_trade` |
| History replay | `GET /api/v1/large-trades/history` | `get_large_trades_history` |
| Resumable event cursor | `GET /api/v1/events/feed/since` | `get_event_replay_since` |
| Largest open positions | `GET /api/v1/large-positions` | `get_large_positions` |

```bash
curl -s 'https://0xinsider.com/sandbox/api/v1/large-trades?limit=25'
```

The feed defaults to `min_size=5000` (USD); `min_grade`, `category` and
`suspicious_only` narrow it. Page with the documented cursor rather than an
offset. The feed moves while you read it, so an offset silently skips or
repeats rows. Responses are cached privately for 30 seconds, so polling faster
than every 30 seconds returns nothing new.

`/api/v1/whale-trades` and its sub-paths, and the `get_whale_trades`,
`get_whale_trade` and `get_whale_trades_history` tools, are deprecated aliases
of these. They answer until 2027-02-01; then the paths return `410 Gone` and
the tools are removed. Write new calls against `/large-trades`.

## Review score

Every feed trade carries `review_score`, from 0.0 to 1.0. It ranks
**attention, not outcomes**. A 0.9 means the trade is unusual enough to look
at; it does not mean the position wins. `signal_score` is its deprecated alias
(removed 2027-02-01).

`review_score` is today's value. `recorded_review_score` is the score written
once when the trade row was inserted; it is null for rows before
2026-08-03T11:59Z and is never backfilled, so a null means "not captured",
not zero. Backtest on `recorded_review_score` and
`trader.grade_at_trade`, never on today's `review_score` or grade.

A suspicious-trade flag's `suspicion_score` (0 to 100, flagged from 60) is a
separate quantity, and the two do not convert. Never present either as a
probability or an expected return.

## Counterparties

A large trade fills against other orders. `GET /api/v1/large-trades/{id}`
embeds the first page in `counterparty_analysis`; two paged sub-resources read
the rest of the same snapshot:

- `GET /api/v1/large-trades/{id}/counterparties/executions` (`get_large_trade_counterparty_executions`)
- `GET /api/v1/large-trades/{id}/counterparties/executions/{execution_id}/makers` (`get_large_trade_counterparty_makers`)

Pass the `snapshot_id` the trade returned. A changed snapshot answers
`cursor_expired`: read the trade again and restart. Follow the cursor to
completion before you total anything; a partial page is a partial total, not a
small one.

## Replay before settlement

`GET /api/v1/large-trades/history` answers the research question: what
large-trade activity did this market see before it resolved? Constrain it by
`condition_id`, `trader` and an RFC 3339 `from`/`to` window rather than pulling
the whole range and filtering client side. It reads 0xinsider's stored record
and reports itself as best effort; do not describe it as complete provider
history.

Use `GET /api/v1/events/feed/since` for a resumable event cursor when you are
driving an alert or an incremental sync. It replays from a position you hold,
so a restart does not drop or duplicate events. Its filters are bound to the
cursor: a cursor issued under other filters answers `cursor_expired`.

## Coverage boundary

The feed covers sports and esports markets only, and it is what 0xinsider
ingests from Polymarket, not complete exchange history. Say "in the tracked
feed" rather than "all trades" when you report a count, and never imply a zero
where the answer is an absence of coverage.

## Real-time

`GET /api/v1/stream` is a resumable SSE stream. Builder webhooks are the push
alternative: create a destination under `/api/v1/webhooks`, verify it, and read
`/api/v1/webhooks/events` for the event catalog. `large_trade_inserted_v2`
delivers one matching large trade per delivery. Rotate the signing secret with
`/api/v1/webhooks/{id}/rotate-secret` and verify every delivery signature. The
MCP server reads destinations with `list_webhooks` and `get_webhook` but cannot
create, change or delete them.

## Linking out

Every link to polymarket.com carries `via=0xinsider` then `r=0xinsidercom`,
after any query parameters it already has, replacing any earlier copy of
either tag: `https://polymarket.com/event/<slug>?via=0xinsider&r=0xinsidercom`.

## Related skills

- `0xinsider-api-access` - credentials, hosts, rate limits, deprecated paths
- `polymarket-wallet-grades` - the wallet behind the trade
- `polymarket-sharp-money` - directional flow across a market
