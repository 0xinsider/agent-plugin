---
name: polymarket-whale-trades
description: Read large Polymarket trades on sports and esports markets through the 0xinsider API or MCP server - list the live feed, replay history, inspect one trade's counterparties, and interpret the significance score. Use when asked which wallets placed the largest trades on a game, to follow large-trade activity on Polymarket, to build a large-trade alert or feed, or to replay a market's large-trade history before it settled.
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

## The four calls

| Need | Call | MCP tool |
| --- | --- | --- |
| Live feed | `GET /api/v1/whale-trades` | `get_whale_trades` |
| One trade | `GET /api/v1/whale-trades/{id}` | `get_whale_trade` |
| History replay | `GET /api/v1/whale-trades/history` | `get_whale_trades_history` |
| Resumable stream | `GET /api/v1/events/feed/since` | `get_event_replay_since` |

```bash
curl -s 'https://0xinsider.com/sandbox/api/v1/whale-trades?limit=25'
```

Page with the documented cursor rather than an offset. The feed moves while you
read it, so an offset silently skips or repeats rows.

## Significance score

Every feed trade carries a normalized significance score from 0.0 to 1.0.

It ranks **attention, not outcomes**. A 0.9 means the trade is unusual enough to
look at; it does not mean the position wins. It is a separate quantity from
Insider Radar's 0-100 review score, and the two are not convertible. Never
present a significance score as a probability or an expected return.

## Counterparties

A large trade fills against other orders. Two paged sub-resources expose who
was on the other side:

- `GET /api/v1/whale-trades/{id}/counterparties/executions`
- `GET /api/v1/whale-trades/{id}/counterparties/executions/{execution_id}/makers`

Both are paged. Follow the cursor to completion before you total anything; a
partial page is a partial total, not a small one.

## Replay before settlement

`GET /api/v1/whale-trades/history` answers the research question: what
large-trade activity did this market see before it resolved? Constrain it by
market and time window rather than pulling the whole range and filtering client
side.

Use `GET /api/v1/events/feed/since` for a resumable event cursor when you are
driving an alert or an incremental sync. It replays from a position you hold,
so a restart does not drop or duplicate events.

## Coverage boundary

The feed is what 0xinsider ingests from Polymarket. It is not complete exchange
history. Say "in the tracked feed" rather than "all trades" when you report a
count, and never imply a zero where the answer is an absence of coverage.

## Real-time

`GET /api/v1/stream` is a resumable SSE stream. Builder webhooks are the push
alternative: create a destination under `/api/v1/webhooks`, verify it, and read
`/api/v1/webhooks/events` for the event catalog. Rotate the signing secret with
`/api/v1/webhooks/{id}/rotate-secret` and verify every delivery signature.

## Linking out

Every link to polymarket.com carries `?r=0xinsidercom`.

## Related skills

- `0xinsider-api-access` - credentials, hosts, rate limits
- `polymarket-wallet-grades` - the wallet behind the trade
- `polymarket-sharp-money` - directional flow across a market
