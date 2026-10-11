---
name: polymarket-sharp-money
description: Read directional sharp-money and smart-money flow on Polymarket sports and esports markets through the 0xinsider API or MCP server, including market intel, live snapshots, the Max-only Edge Hunter board, full ranked signals and cohort observations, and Insider Radar flags. Use when asked how graded-wallet flow is distributed across a market's outcomes, where high-grade wallets are concentrating, how to read a market's flow analytics, or to check review flags on a market.
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
| Edge Hunter quoted board (Max) | `GET /api/v1/edge-hunter` | `get_edge_hunter` (default view) |
| Full ranked signals (Max) | `GET /api/v1/edge-hunter?view=signals` | `get_edge_hunter` (`view: "signals"`) |
| Holder observations (Max) | `GET /api/v1/edge-hunter?view=observations&cohort=wider_holder` | `get_edge_hunter` (`view: "observations"`, `cohort`) |
| Review flags | `GET /api/v1/insider-radar` | `get_insider_radar` |

Market-specific calls use `condition_id`. Resolve a name through
`GET /api/v1/markets/search` before calling them. Edge Hunter returns the
ranked all-sports board or advanced collection without a required market identifier.

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

Use `GET /api/v1/edge-hunter` or the single MCP `get_edge_hunter` tool.
All views require Max; an active Pro credential receives `403 forbidden`
with `error.reason: max_plan_required`. The catalog remains 45 tools.

### Default quoted board

Omit `view` or use `view=board` to preserve the board shown at
https://0xinsider.com/sports/edge. It accepts `horizon_hours` (default 48,
clamp 1..48), uses a B-grade floor across all sports and esports, and returns
at most 30 full ranked rows with `total` before that cap. Advanced category,
cohort, limit, cursor and minimum-grade parameters are rejected for this view.

The REST body is `{object: "edge_hunter", data: <board>, meta: <meta>}`.
Read `data.rows`, `viewer: "max"`, `state: "full"` and `free: false`.
MCP keeps the board payload plus REST meta. `as_of`, `cached_age_seconds`,
`ranking_source` and `directional_source` date the snapshot and its evidence.
Both outcome `current_price` quotes have independent basis and timestamps;
a missing quote is null. The default board has no ETag/304 path.

### Full ranked signals

Select `view=signals` to retain the complete ranked feed, including category
skill and source evidence. Its body is `{object: "list", view: "signals",
data: [...], has_more, next_cursor?, total?, meta}`.

| Parameter | Meaning |
| --- | --- |
| `category` | Optional canonical sport or provider alias; non-sport categories return an empty collection. |
| `min_grade` | S/A/B, default B. Requires at least one piled-side holder at that floor or better; does not remove other holders. |
| `horizon_hours` | Default 12, clamp 1..48. |
| `limit` | Default 20, clamp 1..100. |
| `cursor` | Opaque continuation bound to view, normalized filters, snapshot and order. |

Each record retains `condition_id`, nullable piled-side `token_id`, canonical
and raw category, identity/kickoff, `signal_created_at`/`ranked_at`,
`side`/`piled_side`, provider outcome index, sharp share, dollar backing,
grade counts, holder count, top grade, signed score/share, volume in shares,
conviction/backing scores, one-way/hedged holder evidence, directional
confidence/rank score, `category_skill` and rank. Established synonymous
fields retain the same values. A binary outcome index is not a home/away label.

Category-skill evidence includes status, source/model/taxonomy, writer clocks,
coverage, covered backing and holder edge/concentration measures. It does not
change signal membership or ranking. Read `meta.ranking_source`,
`meta.directional_source` and category-skill provenance; `db_only`, `degraded`,
partial and unknown values must remain explicit.

### Cohort observations

Select `view=observations` and require `cohort=wider_holder|in_play|emerging_pile`.
Optional queries are `category`, `limit` (default 20, clamp 1..100) and `cursor`;
`horizon_hours` and `min_grade` are rejected. The pre-game universe is 48 hours;
in-play kickoff lookback is 12 hours.

The full body retains `object: "list"`, `view: "observations"`, rows in `data`,
`has_more`, always-present nullable `next_cursor`, `snapshot_as_of`,
`degraded`, `funnel` and `meta`. Rows carry verified non-null `token_id`,
identity/category/event/parent/kickoff, `observed_at`, cohort,
`observation_only: true`, side/index, `backed_price`, all grade/backing/score
aliases, provider read source, holder scan completeness/clock, directional
status and one-way/hedged evidence, volume in shares and rank.

`backed_price` is the observed aggregate market midpoint; it is not a wallet
entry cost. In-play snapshots are reusable for at most 30 seconds and fail
closed without fresh provider-live, holder and directional evidence.
Wider-holder may retain explicit unknown direction. Emerging pile is a
wider-holder projection with sharp_pct in [0.75,0.85), complete holder scans
and future kickoff at a pinned cutoff; it is neither a separate population
nor arrival history.

Read the accountable `funnel.sports` for all 14 sports: board input and
live/upcoming availability, upcoming status/unavailable scopes, diagnostic
input, the closed 25-value terminal map, terminal_total and reconciled.
Every input has one terminal and input equals terminal_total. Snapshot-wide
`degraded` can include unknown completeness beyond terminal accounting;
intentional bounded capacity_limited alone does not imply degradation.

### Typed clients, conditional reads and migration

SDK 0.21.0 literal view calls narrow to the corresponding full envelope:

```ts
const board = await client.edgeHunter.get();
const signals = await client.edgeHunter.signals({ category: "Soccer", limit: 20 });
const observations = await client.edgeHunter.observations({ cohort: "in_play" });
const conditional = await client.edgeHunter.getConditional(
  { view: "observations", cohort: "in_play" },
  { headers: { "If-None-Match": observations.meta.etag ?? "" } },
);
```

Shared `client.list()`, `paginate()`, `paginatePages()` and `collect()` also
accept `getEdgeHunter` with an explicit advanced query. They preserve page
and row types, continuation progress, resume information and cancellation.
MCP 3.1.0 uses one `get_edge_hunter` selector and retains every advanced list,
page, snapshot, funnel and REST-meta field. Its default board output is stable.

Signals and observations support semantic ETag/If-None-Match/304; the default
board does not. Both advanced views require private revalidation and read
warmed generations only. Missing/stale readiness returns 503
`read_model_warming` with `Retry-After: 5`; requests do not compute holders
or call providers. Old route
cursors cannot be reused, changed filters/views cannot share a cursor, and a
refreshed snapshot returns `cursor_expired`: restart at page one. All four
legacy sports URLs/tools remain retired; migrate to the matching advanced
view rather than restoring their calls.

Follow the current typed contract at
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
