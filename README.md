# 0xinsider Agent Plugin

[![skills.sh](https://skills.sh/b/0xinsider/agent-plugin)](https://skills.sh/0xinsider/agent-plugin)

The official [Agent Plugin](https://agent-plugins.org/specification) for
**0xinsider** - Polymarket analytics & infrastructure for sports and esports markets.

It bundles the 0xinsider MCP server with five skills that teach an agent how to
read large trades, wallet grades, sharp-money flow, and published research
without misreporting them.

## Install

```bash
npx skills add 0xinsider/agent-plugin
```

Refresh an existing installation with:

```bash
npx skills@latest update 0xinsider-api-access polymarket-whale-trades polymarket-wallet-grades polymarket-sharp-money polymarket-market-research
```

Or install one skill without adding the whole plugin:

```bash
npx skills add 0xinsider/agent-plugin --skill polymarket-wallet-grades
```

## Contents

| Skill | Covers |
| --- | --- |
| [`0xinsider-api-access`](skills/0xinsider-api-access/SKILL.md) | Credentials, sandbox vs live hosts, MCP transports, rate limits, deprecated paths, typed errors |
| [`polymarket-whale-trades`](skills/polymarket-whale-trades/SKILL.md) | Large-trade feed, history replay, counterparties, review score |
| [`polymarket-wallet-grades`](skills/polymarket-wallet-grades/SKILL.md) | Trader grades, sports and esports P&L, category records, leaderboards, position timelines, exports |
| [`polymarket-sharp-money`](skills/polymarket-sharp-money/SKILL.md) | Market flow, graded holders, candles, Max-only Edge Hunter board/full signals/cohort observations, suspicious-trade flags |
| [`polymarket-market-research`](skills/polymarket-market-research/SKILL.md) | Market search and explore, games, published research, report snapshots, Pick of the Day, coverage |

Plugin 1.3.0 re-syncs every skill with the live API and MCP server. Calls use
the canonical paths and tools (`/large-trades`, `/suspicious-trades`,
`/market/{condition_id}/flow`, `/markets/flow/batch`,
`/markets/sharp-money-flows`, `/coverage`); the older names are listed only as
deprecated aliases, which stop working on 2027-02-01. A wallet's P&L is
`pnl.in_scope.realized_pnl`, because `pnl.realized` has not been sent since
2026-10-27. The large-trade score is `review_score`, `{address}` accepts
usernames, exports start with a POST, rate limits are per plan (100 a minute
on Pro, 200 on Max), polymarket.com links carry `via=0xinsider` then
`r=0xinsidercom`, and `mcp.json` now validates against the Agent Plugins 1.0.0
MCP schema and points at the canonical endpoint
`https://api.0xinsider.com/api/v1/mcp`.

Plugin 1.2.0 documents configurable selections across every Edge Hunter view:
sports/esports, canonical sports, exact leagues, market types, markets, events
and series. Board defaults remain compatible; rows add frozen `market_context`
for authoritative selector discovery, and selected responses echo normalized
`selection`. Advanced records retain pagination, evidence and conditional reads.
Selections need TypeScript SDK 0.22.0 or later and CLI/MCP runtime 3.2.0 or later.
The four legacy URLs/tools remain retired.

[`AGENTS.md`](AGENTS.md) holds the rules for AI coding agents writing
integration code against the API. `.cursorrules` mirrors it.

## MCP server

[`mcp.json`](mcp.json) points at the remote Streamable HTTP endpoint:

```json
{
  "mcpServers": {
    "0xinsider": {
      "type": "streamable-http",
      "url": "https://api.0xinsider.com/api/v1/mcp"
    }
  }
}
```

It carries no credential. Agent Plugins clients do not expand environment
variables in headers, so the client signs in on the first `tools/call`: the
`401` names the OAuth 2.1 protected-resource metadata, and the authorization
server supports dynamic client registration and PKCE. To use an API key
instead, add the server to your client's own MCP configuration. In clients
that expand `${VAR}` in headers, such as Claude Code's `.mcp.json`:

```json
{
  "mcpServers": {
    "0xinsider": {
      "type": "http",
      "url": "https://api.0xinsider.com/api/v1/mcp",
      "headers": { "Authorization": "Bearer ${OXINSIDER_API_KEY}" }
    }
  }
}
```

The server exposes read-only tools across traders, markets, games, large
trades, positions, flow, research, reports, and webhooks: 55 on 2026-10-28,
nine of them deprecated aliases. `initialize`, `ping` and `tools/list` need no
credential, so `tools/list` is always the current catalog; `tools/call` takes
an API key or an OAuth 2.1 token. Its card is at
[`/.well-known/mcp`](https://0xinsider.com/.well-known/mcp).
`https://0xinsider.com/mcp` answers with the same tools, but OAuth clients
should use the endpoint above, which the protected-resource metadata names.

A local stdio path ships in the CLI:

```bash
npx -y @0xinsider/mcp@latest init
```

## Try it with no account

Every documented operation answers from the sandbox with no credential:

```bash
curl -s https://0xinsider.com/sandbox/api/v1/leaderboard
```

One POST returns a sandbox key and the steps to live access:

```bash
curl -s -X POST https://api.0xinsider.com/api/v1/agents/register \
  -H 'Content-Type: application/json' -d '{"name":"my-agent"}'
```

## Official resources

- Developer portal: <https://0xinsider.com/developers>
- API documentation: <https://docs.0xinsider.com>
- OpenAPI 3.1 spec: <https://0xinsider.com/api/v1/openapi.json>
- Machine-readable index: <https://0xinsider.com/llms.txt>
- Python SDK: <https://github.com/0xinsider/0xinsider-python>
- Go SDK: <https://github.com/0xinsider/0xinsider-go>
- Node.js and TypeScript SDK: <https://github.com/0xinsider/0xinsider-node>
- Rust SDK (git dependency; crates.io release pending): <https://github.com/0xinsider/0xinsider-rust>
- CLI and MCP package: <https://www.npmjs.com/package/@0xinsider/mcp>

## License

MIT
