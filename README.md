# 0xinsider Agent Plugin

[![skills.sh](https://skills.sh/b/0xinsider/agent-plugin)](https://skills.sh/0xinsider/agent-plugin)

The official [Agent Plugin](https://agent-plugins.org/specification) for
**0xinsider** - Polymarket analytics for sports and esports in real-time.

It bundles the 0xinsider MCP server with five skills that teach an agent how to
read large trades, wallet grades, sharp-money flow, and published research
without misreporting them.

## Install

```bash
npx skills add 0xinsider/agent-plugin
```

Or install one skill without adding the whole plugin:

```bash
npx skills use 0xinsider/agent-plugin@0xinsider-api-access
```

## Contents

| Skill | Covers |
| --- | --- |
| [`0xinsider-api-access`](skills/0xinsider-api-access/SKILL.md) | Credentials, sandbox vs live hosts, MCP transports, rate limits, typed errors |
| [`polymarket-whale-trades`](skills/polymarket-whale-trades/SKILL.md) | Large-trade feed, history replay, counterparties, significance score |
| [`polymarket-wallet-grades`](skills/polymarket-wallet-grades/SKILL.md) | Trader grades, settled P&L and its trust metadata, leaderboards, position timelines |
| [`polymarket-sharp-money`](skills/polymarket-sharp-money/SKILL.md) | Directional flow, market intel, sports-edge signals, Insider Radar flags |
| [`polymarket-market-research`](skills/polymarket-market-research/SKILL.md) | Market search and explore, published research, report snapshots |

[`AGENTS.md`](AGENTS.md) holds the rules for AI coding agents writing
integration code against the API. `.cursorrules` mirrors it.

## MCP server

[`mcp.json`](mcp.json) points at the remote Streamable HTTP endpoint:

```json
{
  "mcpServers": {
    "0xinsider": {
      "type": "http",
      "url": "https://0xinsider.com/mcp",
      "headers": { "Authorization": "Bearer ${OXINSIDER_API_KEY}" }
    }
  }
}
```

The server exposes 33 tools across traders, markets, large trades, positions,
flow, research, reports, and webhooks. Its card is at
[`/.well-known/mcp`](https://0xinsider.com/.well-known/mcp).

A local stdio path ships in the CLI:

```bash
npx -y 0xinsider init
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
- CLI and MCP package: <https://www.npmjs.com/package/@0xinsider/mcp>

## License

MIT
