---
name: polymarket-sharp-money
description: Read directional sharp-money flow on Polymarket sports and esports markets through the 0xinsider API or MCP server, including market flow, graded holders, live snapshots, price candles, the configurable Max-only Edge Hunter board, full ranked signals and cohort observations, and suspicious-trade flags (formerly Insider Radar). Use when asked how graded-wallet flow is distributed across a market's outcomes, where high-grade wallets are concentrating, which graded wallets hold each side of a game, how to read a market's flow analytics (sometimes called smart-money flow), or to check suspicious-trade flags on a market.
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
| One market | `GET /api/v1/market/{condition_id}/flow` | `get_market_flow` |
| Many markets | `POST /api/v1/markets/flow/batch` | `batch_get_market_flow` |
| Graded holders of a market | `GET /api/v1/market/{condition_id}/holders` | `get_market_holders` |
| Live snapshot | `GET /api/v1/market/{condition_id}/snapshot` | `get_market_snapshot` |
| OHLC candles | `GET /api/v1/market/{condition_id}/candles` | `get_market_candles` |
| Edge Hunter quoted board (Max) | `GET /api/v1/edge-hunter` | `get_edge_hunter` (default view) |
| Full ranked signals (Max) | `GET /api/v1/edge-hunter?view=signals` | `get_edge_hunter` (`view: "signals"`) |
| Holder observations (Max) | `GET /api/v1/edge-hunter?view=observations&cohort=wider_holder` | `get_edge_hunter` (`view: "observations"`, `cohort`) |
| Suspicious-trade flags | `GET /api/v1/suspicious-trades` | `get_suspicious_trades` |
| One flag | `GET /api/v1/suspicious-trades/{id}` | `get_suspicious_trade` |

Market-specific calls use `condition_id`. Resolve a name through
`GET /api/v1/markets/search` before calling them. Edge Hunter returns the
ranked all-sports board or advanced collection without a required market identifier.

Deprecated aliases answer until 2027-02-01, then return `410 Gone` and lose
their MCP tools: `/api/v1/market/{condition_id}/intel` (`get_market_intel`),
`/api/v1/markets/intel/batch` (`batch_get_market_intel`),
`/api/v1/markets/smart-money-flows` (`get_smart_money_flows`) and
`/api/v1/insider-radar` (`get_insider_radar`, `get_insider_radar_flag`). Write
new calls against the paths in the table.

`holders` is the graded roster of one market: every S, A and B wallet with
open shares on either outcome, from a complete provider holder scan, listed
once on its net side. Read `scan.wallet_count` for the whole population; do
not page `/api/v1/positions` to rebuild it.

## The sign convention

Market flow reports outcome-aware large-trade flow; read its `sharp_money`
object (`smart_money` is a deprecated alias with the same bytes). Getting the
sign wrong inverts the conclusion, so it is worth stating plainly:

- `BUY YES` **adds** exposure
- `SELL NO` **adds** exposure
- `BUY NO` **subtracts** exposure
- `SELL YES` **subtracts** exposure

A zero-flow YES tie-break is not conviction. When flow nets to zero, the YES
label is a deterministic tie-break, and reporting it as a lean toward YES is a
fabrication.

## Suspicious-trade flags

Formerly Insider Radar. `GET /api/v1/suspicious-trades` lists stored trades
whose recorded `suspicion_score` (0 to 100) met the live flag floor of 60,
highest first. `min_suspicion` raises the floor. `mode=stable` pins a paginated
walk to one published scoring generation and returns `cursor_expired` when
scores or filters change; the default `mode=live` serves cached pages. The
compatible `severity=watch` filter stays empty until a watch policy exists.

A flag does not establish intent, non-public knowledge, or a future outcome.
It is a pointer to something worth a human look. The 0-100 `suspicion_score`
is a different quantity from a large trade's 0.0-1.0 `review_score`; do not
convert or compare them.

## Edge Hunter

Use `GET /api/v1/edge-hunter` or the single MCP `get_edge_hunter` tool.
All views require Max; an active Pro credential receives `403 forbidden`
with `error.reason: max_plan_required`.

### Default quoted board

Omit `view` or use `view=board` to preserve the board shown at
https://0xinsider.com/sports/edge. It accepts `horizon_hours` (default 48,
clamp 1..48). Defaults are B grade floor, all sports/esports and at most 30
full rows. Common selections, legacy `category`, `min_grade` S/A/B and
`limit` (default 30, clamp 1..30) are configurable. `total` counts selected
admitted rows before the cap. Cohort and cursor are rejected for this view.

The REST body is `{object: "edge_hunter", data: <board>, meta: <meta>}`.
Read `data.rows`, `viewer: "max"`, `state: "full"` and `free: false`.
MCP keeps the board fields at the root of `structuredContent`, beside REST
`meta` and optional `selection`. `as_of`, `cached_age_seconds`,
`ranking_source` and `directional_source` date the snapshot and its evidence.
Both outcome `current_price` quotes have independent basis and timestamps;
a missing quote is null. The default board has no ETag/304 path.

### Common selections and discovery

Every view accepts the same selection axes:

| Selection | Meaning |
| --- | --- |
| `kind` | `sports` or `esports`; omission covers both |
| `sports` | Canonical buckets such as `Football`, `Soccer`, `Esports` or `Table Tennis`; case-insensitive |
| `leagues` | Normalized machine/provider keys such as `nfl`, `epl`, `cs2` or `lol` |
| `market_types` | Exact provider `sports_market_type` values |
| `condition_ids` | Exact market IDs, raw or `mkt_`-prefixed |
| `event_slugs` / `series_slugs` | Exact provider event/series identities |
| `category` | Legacy bucket alias: `NFL` means Football; use `leagues=nfl` for NFL only |

REST repeats bare query names or uses CSV within a value; SDK/MCP use arrays.
Each list has 1..100 supplied values before deduplication. OR within a list,
AND across axes. Known machine/provider league aliases normalize; unknown keys
match exact lowercase identities. Market types and event/series slugs are
trimmed case-sensitive provider strings. Unknown sports are rejected; unknown
provider identities and market IDs can produce empty selected results.

```ts
const nfl = await client.edgeHunter.get({ kind: "sports", leagues: ["nfl"], limit: 10 });
const soccer = await client.edgeHunter.signals({ sports: ["Soccer"], market_types: ["moneyline"] });
const liveEsports = await client.edgeHunter.observations({ cohort: "in_play", kind: "esports", leagues: ["cs2", "lol"] });
```

Selected 200 responses echo requested normalized axes in top-level `selection`;
SDK/MCP force strict query validation and refuse a missing echo. Empty lists are
invalid. Blank category retains omission behavior. Each row adds required
nullable `market_context` with required nullable `league`, `series_slug`,
`market_type` and `metadata_updated_at`. These frozen facts are authoritative
selector values. The timestamp is the local canonical metadata writer clock,
not provider observation time. Missing selected facts exclude rows. Legacy
unselected cache generations can carry context:null; selected reads without a
ready sidecar return warming, never a silently widened result.

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
It accepts the common selections, `category`, `limit` (default 20, clamp 1..100) and `cursor`;
`horizon_hours` and `min_grade` are rejected. The pre-game universe is 48 hours;
in-play kickoff lookback is 12 hours.

With `cohort=wider_holder`, `admission=most_held_48h` opts into the most-held
selection instead of the default one: per sport, up to 8 markets kicking off
within 48 hours, most graded holders first. The response then echoes
`admission`, and its funnel, coverage and degradation fields describe that
selection. The parameter is rejected for every other view and cohort.

The full body retains `object: "list"`, `view: "observations"`, rows in `data`,
`has_more`, always-present nullable `next_cursor`, `snapshot_as_of`,
`degraded`, `degradation_reasons` (empty exactly when `degraded` is false),
`coverage`, `funnel` and `meta`. Wider-holder reads add `departures`: markets
whose card left the selection before kickoff, with the reason. Each row names
the place that admitted it in `admitted_by`. Rows carry verified non-null `token_id`,
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

Funnel accounting describes the source generation/category, not a selected
league/type/market input census. Read `funnel.sports` for all 14 sports: board input and
live/upcoming availability, upcoming status/unavailable scopes, diagnostic
input, the closed 25-value terminal map, terminal_total and reconciled.
Every input has one terminal and input equals terminal_total. Snapshot-wide
`degraded` can include unknown completeness beyond terminal accounting;
intentional bounded capacity_limited alone does not imply degradation.

### Typed clients, conditional reads and migration

SDK 0.22.0 literal view calls narrow to the corresponding full envelope:

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
MCP 3.2.0 uses one `get_edge_hunter` selector and retains every advanced list,
page, snapshot, funnel and REST-meta field. Its default board behavior stays compatible with additive row context and
optional normalized selection echo.

Signals and observations support semantic ETag/If-None-Match/304; the default
board does not. Both advanced views require private revalidation; all public
views read warmed generations only. Missing/stale readiness returns 503
`read_model_warming` with `Retry-After: 5` (45 while a sport's cold pregame
holder reads restart); requests do not compute holders
or call providers. Old route
cursors cannot be reused, changed filters/views cannot share a cursor, and a
refreshed snapshot returns `cursor_expired`: restart at page one. All four
legacy sports URLs/tools remain retired: `/api/v1/sports-edge-signals`,
`/api/v1/sports-edge-observations`, `/api/v1/sports/pre-game-sides` and
`/api/v1/sports/pre-game-side-observations` return `410 endpoint_retired`.
Migrate to the matching advanced view rather than restoring their calls.

Follow the current typed contract at
https://docs.0xinsider.com/api-reference/endpoint/get-edge-hunter.

## Reporting flow without overclaiming

Flow is what graded wallets did. It is analytics, not a forecast and not
advice. Report
the direction, the size, the window it covers, and the coverage basis. Edge
Hunter states its basis in `meta.ranking_source`, `meta.directional_source`
and the category-skill coverage (`partial_whale_threshold_fills` or
`graded_wallet_fills`). If the basis is partial or degraded, say so in the
same sentence as the number.

Keep full numeric precision until the final render.

## Related skills

- `0xinsider-api-access` - credentials, hosts, rate limits
- `polymarket-whale-trades` - the individual fills behind the flow
- `polymarket-wallet-grades` - who is in the cohort
