---
name: polymarket-market-research
description: Search Polymarket sports and esports markets and 0xinsider's published research, explore markets by category, list games with both sides and their linked markets, pull daily, weekly or monthly report snapshots, and read the Pick of the Day record through the API or MCP server. Use when asked to find a Polymarket market or game, search 0xinsider research or learn guides, browse markets by sport, produce a periodic market report, check what the API covers, or check the published pick record.
license: MIT
metadata:
  publisher: 0xinsider
  homepage: https://0xinsider.com/developers
  openapi: https://0xinsider.com/api/v1/openapi.json
---

# Polymarket market research

Finding markets and games, and finding what has been published about them.
Read `0xinsider-api-access` first for credentials and hosts.

## Search and explore

| Need | Call | MCP tool |
| --- | --- | --- |
| Find a market | `GET /api/v1/markets/search` | `search_markets` |
| Browse by category | `GET /api/v1/markets/explore` | `explore_markets` |
| Games, both sides | `GET /api/v1/games` | `list_games` |
| One game and its markets | `GET /api/v1/games/{event_slug}` | `get_game` |
| Find published research | `GET /api/v1/content/search` | `search_content` |

```bash
curl -s 'https://0xinsider.com/sandbox/api/v1/markets/search?q=super%20bowl'
```

`markets/search` is the entry point for every other market call: it returns the
`condition_id` that flow, holder, snapshot, and candle endpoints require.

`games` returns one coherent game per row: both sides with their provider IDs
and live scores, the UTC kickoff, the provider's status, and every linked
Polymarket market with its `condition_id`. Use it instead of assembling a game
from market searches and team-name matching. Filter with `sport`, `league`,
`status` and a `starts_after`/`starts_before` window. Soccer 1X2 markets carry
`side` `home`, `draw` or `away`; never treat the draw leg as a competitor.

`content/search` covers published research and learn guides. It returns
canonical links and freshness metadata. Cite the canonical link rather than
paraphrasing a study without attribution, and check the freshness field before
you describe a finding as current.

## Reports

| Granularity | Call | MCP tool |
| --- | --- | --- |
| Selector | `GET /api/v1/reports` | `get_report` |
| Daily | `GET /api/v1/reports/daily` | `get_daily_report_snapshot` |
| Weekly | `GET /api/v1/reports/weekly` | `get_weekly_report_snapshot` |
| Monthly | `GET /api/v1/reports/monthly` | `get_monthly_report_snapshot` |

These are snapshots. A snapshot is the state at its stated timestamp, not a
live read. Quote the timestamp whenever you quote the numbers, and check
`snapshot.status` and `completeness.status` before you treat a range as final.

## Pick of the Day

| Need | Call | MCP tool |
| --- | --- | --- |
| Today's picks | `GET /api/v1/pick-of-the-day` | `get_pick_of_the_day` |
| Track record | `GET /api/v1/pick-of-the-day/archive` | `get_pick_of_the_day_archive` |
| One pick's commitment | `GET /api/v1/pick-of-the-day/ledger/{pick_id}` | `get_pick_of_the_day_ledger_entry` |

Picks are published on a public record, win or lose. Each archived pick
carries its `outcome` (`win`, `loss`, `void` or `pending`); report a past pick
with that result, not the side alone. The ledger (`/api/v1/pick-of-the-day/ledger`)
needs no credential and holds the pre-game commitment for every published
pick. A `404 pick_not_released` means no pick is out yet: schedule one request
for its `error.retry_at` rather than polling.

## Coverage

`GET /api/v1/coverage` (`get_coverage`) needs no credential and declares which
data surfaces the API serves for Polymarket. Coverage is sports and esports
markets only: a market outside them answers `404 not_found` with
`error.reason` set to `market_out_of_scope`, which is terminal, so do not
retry it. Check coverage before you assume a sport, a market type, or a field
is covered. An absent capability is an absence of coverage, not a zero.
`/api/v1/platforms` (`get_platforms`) is its deprecated alias until
2027-02-01.

## Usage

UTC-day usage combines finalized rollups with a disjoint raw tail through
request time. Read it from `GET /api/v1/usage`, which does not spend primary
request quota.

## Linking out

Every link to polymarket.com carries `via=0xinsider` then `r=0xinsidercom`,
after any query parameters it already has, replacing any earlier copy of
either tag: `https://polymarket.com/event/<slug>?via=0xinsider&r=0xinsidercom`.

## Related skills

- `0xinsider-api-access` - credentials, hosts, rate limits
- `polymarket-sharp-money` - flow on a market you found
- `polymarket-wallet-grades` - the wallets behind a position
