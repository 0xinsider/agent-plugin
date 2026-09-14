---
name: polymarket-wallet-grades
description: Look up a Polymarket wallet's grade, settled P&L, positions and history through the 0xinsider API or MCP server, and read the trust metadata that says when a P&L figure is computed rather than inferred. Use when asked how good a Polymarket trader is, to grade a wallet, to compare traders, to pull a leaderboard or trending wallets, or to check a wallet's position timeline on a market.
license: MIT
metadata:
  publisher: 0xinsider
  homepage: https://0xinsider.com/developers
  openapi: https://0xinsider.com/api/v1/openapi.json
---

# Polymarket wallet grades

Grades and P&L for tracked Polymarket wallets. Read `0xinsider-api-access`
first for credentials and hosts.

## Lookups

| Need | Call | MCP tool |
| --- | --- | --- |
| One trader | `GET /api/v1/trader/{address}` | `get_trader` |
| Many traders | `POST /api/v1/traders/batch` | `batch_get_traders` |
| Ranked list | `GET /api/v1/leaderboard` | `get_leaderboard` |
| Rising wallets | `GET /api/v1/leaderboard/trending` | `get_trending_wallets` |
| P&L series | `GET /api/v1/trader/{address}/pnl` | `get_trader_pnl` |
| Timeline on a market | `GET /api/v1/trader/{address}/position-timeline` | `get_position_timeline` |
| Prose context | `GET /api/v1/trader/{address}/context.md` | - |

Batch reads accept 25 addresses per call. Use `POST /api/v1/traders/batch`
instead of a loop: the loop burns 25 requests against a 100-per-minute limit to
fetch what one call returns.

```bash
curl -s https://0xinsider.com/sandbox/api/v1/leaderboard
```

`context.md` returns the trader as Markdown prose rather than JSON. It is the
cheaper input when you are handing a wallet to a language model instead of to
code.

## Identity

`/api/v1/trader/{address}` is an address lookup. A username lookup takes
`@name` through the unified resolver at
`/api/v1/traders/{trader}/position-timeline`. Sending a username where an
address is expected returns a lookup failure, not an empty trader.

## Reading P&L honestly

This is the part that gets misreported.

`pnl.realized` carries trust metadata. With `expand=trust`, a matching native
accounting snapshot marks the figure **computed**: native Polymarket realized
P&L plus credited maker and taker rebates, fees already included.

Without a matching native snapshot, `pnl.realized` is **omitted** and its trust
metadata is unavailable.

When it is omitted, report it as unavailable. Raw total P&L is never a
substitute for realized P&L, and substituting it produces a number that looks
authoritative and is wrong.

## Coverage boundary

Grades and P&L apply to tracked Polymarket wallets with enough history. An
ungraded wallet is uncovered, not unskilled. Do not render an absent grade as a
low grade.

Position timelines expose outcome-specific running fill totals across pages.
They are not complete holdings: split, merge, redemption and negative-risk
activity are excluded. Follow the cursor to the end before totaling.

## Naming

"Profitable wallets" is the name of the sharp cohort. "Tracked wallet" means
all-grades coverage. The two are not interchangeable.

## Exports

`GET /api/v1/trader/{address}/export` starts a snapshot job. Poll
`/export/status`, then fetch `/export/download`. It is an async job: do not
block on the first call, and do not re-issue the job when a poll is pending.

## Related skills

- `0xinsider-api-access` - credentials, hosts, rate limits
- `polymarket-whale-trades` - what the wallet just did
- `polymarket-sharp-money` - the cohort view across a market
