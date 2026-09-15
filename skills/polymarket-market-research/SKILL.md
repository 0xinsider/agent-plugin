---
name: polymarket-market-research
description: Search Polymarket sports and esports markets and 0xinsider's published research, explore markets by category, and pull daily, weekly or monthly report snapshots through the API or MCP server. Use when asked to find a Polymarket market, search 0xinsider research or learn guides, browse markets by sport, or produce a periodic market report.
license: MIT
metadata:
  publisher: 0xinsider
  homepage: https://0xinsider.com/developers
  openapi: https://0xinsider.com/api/v1/openapi.json
---

# Polymarket market research

Finding markets, and finding what has been published about them. Read
`0xinsider-api-access` first for credentials and hosts.

## Search and explore

| Need | Call | MCP tool |
| --- | --- | --- |
| Find a market | `GET /api/v1/markets/search` | `search_markets` |
| Browse by category | `GET /api/v1/markets/explore` | `explore_markets` |
| Find published research | `GET /api/v1/content/search` | `search_content` |

```bash
curl -s 'https://0xinsider.com/sandbox/api/v1/markets/search?q=super%20bowl'
```

`markets/search` is the entry point for every other market call: it returns the
`condition_id` that intel, snapshot, and candle endpoints require.

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
live read. Quote the timestamp whenever you quote the numbers.

## Platform coverage

`GET /api/v1/platforms` returns the capability matrix. Check it before you
assume a sport, a market type, or a field is covered. An absent capability is
an absence of coverage, not a zero.

## Usage

UTC-day usage combines finalized rollups with a disjoint raw tail through
request time. Read it from `GET /api/v1/usage`, which does not spend primary
request quota.

## Linking out

Every link to polymarket.com carries `?r=0xinsidercom`.

## Related skills

- `0xinsider-api-access` - credentials, hosts, rate limits
- `polymarket-sharp-money` - flow on a market you found
- `polymarket-wallet-grades` - the wallets behind a position
