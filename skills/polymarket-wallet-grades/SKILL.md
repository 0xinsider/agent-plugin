---
name: polymarket-wallet-grades
description: Look up a Polymarket wallet's grade, sports and esports P&L, category record, positions and history through the 0xinsider API or MCP server, and read the fields that say when a P&L figure is not computed yet. Use when asked how good a Polymarket trader is, to grade a wallet, to compare traders, to pull a leaderboard or trending wallets, to check a wallet's record in one sport, its grade at a past date, or its position timeline on a market.
license: MIT
metadata:
  publisher: 0xinsider
  homepage: https://0xinsider.com/developers
  openapi: https://0xinsider.com/api/v1/openapi.json
---

# Polymarket wallet grades

Grades S to F and sports and esports P&L for tracked Polymarket wallets,
computed from settled history. Read `0xinsider-api-access` first for
credentials and hosts.

## Lookups

| Need | Call | MCP tool |
| --- | --- | --- |
| One trader | `GET /api/v1/trader/{address}` | `get_trader` |
| Many traders | `POST /api/v1/traders/batch` | `batch_get_traders` |
| Ranked list | `GET /api/v1/leaderboard` | `get_leaderboard` |
| Rising wallets | `GET /api/v1/leaderboard/trending` | `get_trending_wallets` |
| P&L series | `GET /api/v1/trader/{address}/pnl` | `get_trader_pnl` |
| Record per sport | `GET /api/v1/trader/{address}/categories` | `get_trader_categories` |
| Grade at a past instant | `GET /api/v1/trader/{address}/grade-at?as_of=<RFC3339>` | `get_trader_grade_at` |
| One-call briefing | `GET /api/v1/trader/{address}/context` | `get_trader_context` |
| Briefing as Markdown | `GET /api/v1/trader/{address}/context.md` | - |
| What a wallet holds | `GET /api/v1/positions?wallet={address}` | `get_positions` (`wallet`) |
| Timeline on a market | `GET /api/v1/trader/{address}/position-timeline` | `get_position_timeline` |
| Timeline by trader ID | `GET /api/v1/traders/{trader}/position-timeline` | `get_position_timeline_by_id` |

Batch reads accept 25 addresses or usernames per call. Use
`POST /api/v1/traders/batch` instead of a loop: the loop burns 25 requests
against a 100-per-minute Pro limit to fetch what one call returns.

```bash
curl -s https://0xinsider.com/sandbox/api/v1/leaderboard
```

`context.md` returns the trader as Markdown prose rather than JSON. It is the
cheaper input when you are handing a wallet to a language model instead of to
code.

## Identity

`{address}` on `/api/v1/trader/{address}` and its sub-paths takes a `0x`
wallet address, a known username, or a `trd_`-prefixed trader ID that the API
emitted. `/api/v1/traders/{trader}/position-timeline` also takes a bare integer
ID and `@username`, which forces a username lookup for an all-digit name.

A wallet address 0xinsider does not track yet returns `sync_status: unknown`:
it holds no data for that wallet, so check again later, not in a tight loop. A
username or trader ID that matches no trader returns `404 not_found`; do not
retry it.

## Reading P&L honestly

This is the part that gets misreported.

Since 2026-10-27 a wallet's figures cover its sports and esports markets only,
and its realized P&L is `pnl.in_scope.realized_pnl` (on a leaderboard row,
`in_scope.realized_pnl`).

- `pnl.in_scope` is omitted until 0xinsider has computed the wallet's record.
  Absent means not computed yet, never zero.
- `realized_pnl` is null when its `basis` has no published figure to split.
- Compare figures only within one `basis`. A new `basis` value means the rule
  changed.
- Quote `in_scope.as_of` with the number. `markets_out_of_scope` counts the
  wallet's other markets, which 0xinsider no longer lists.
- `pnl.unrealized` is the open-position figure, a separate quantity.

The wallet-wide `pnl.realized`, `pnl.total`, `pnl.net_total`,
`pnl.correction` and `pnl.exact` (and the leaderboard's `pnl`, `realized_pnl`
and `pnl_correction`) were retired on 2026-10-27 and are never sent. Do not
substitute any other figure for a missing `in_scope.realized_pnl`; report it
as not computed yet. Substituting produces a number that looks authoritative
and is wrong.

`GET /api/v1/trader/{address}/pnl` returns the daily series. While its
`series_status` is `pending`, every section is empty and the empty sections
never mean zero. Its days plus `realized.undated_pnl` equal the realized
figure.

With `expand=trust`, each field reports where it came from and whether it is
computed. The retired P&L fields keep their keys there and read
`not_computed`.

## Grades over time

The current grade and the leaderboard rank are today's values. To ask what a
wallet's grade was when it traded, call `grade-at` with that instant. Use its
`grade` only when `status` is `graded`; `unknown` means no recorded
observation, not an ungraded wallet. Never project a current grade backward.

## Coverage boundary

Grades and P&L apply to tracked Polymarket wallets with enough history. An
ungraded wallet is uncovered, not unskilled. Do not render an absent grade as a
low grade.

Position timelines expose outcome-specific running fill totals across pages.
Stored fills start 2026-07-17, and only the wallets 0xinsider syncs most often
have a timeline; other traders return 404. They are not complete holdings:
split, merge, redemption and negative-risk activity are excluded. Follow the
cursor to the end before totaling.

## Naming

"Profitable wallets" is the name of the sharp cohort. "Tracked wallet" means
all-grades coverage. The two are not interchangeable.

## Exports

`POST /api/v1/trader/{address}/export?format=json|ndjson|csv` queues an async
export of the wallet's full dataset and returns a job. Poll
`/export/status?job_id=<id>` until `terminal` is true, then fetch
`/export/download?job_id=<id>` while the status is `ready` (24 hours from
submit). A second POST without `fresh=true` returns the live job rather than
queueing another, so do not loop on submit. `POST /export/cancel?job_id=<id>`
stops a job you no longer want.

`GET /api/v1/trader/{address}/export` (`get_trader_export_snapshot`) returns
the export's source range, completeness and size estimate. It does not start a
job and does not return the data.

## Related skills

- `0xinsider-api-access` - credentials, hosts, rate limits
- `polymarket-whale-trades` - what the wallet just did
- `polymarket-sharp-money` - the cohort view across a market
