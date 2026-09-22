# AGENTS.md - 0xinsider

Instructions for AI coding agents building against 0xinsider. 0xinsider is
Polymarket analytics for sports and esports in real-time: large trades, wallet
grades, sharp-money flow, positions, and research.

This file is the contract for agents writing integration code. Human-facing
documentation lives at https://docs.0xinsider.com.

## What this repository is

The official 0xinsider Agent Plugin: a `plugin.json` manifest, an `mcp.json`
server definition, and five skills under `skills/`. Install it with:

```bash
npx skills add 0xinsider/agent-plugin
```

There is no build step and no runtime here. Everything is Markdown and JSON.

## Start here, in this order

1. `skills/0xinsider-api-access/SKILL.md` - credentials, hosts, rate limits,
   errors. Read this before any other skill.
2. The skill matching the task: `polymarket-whale-trades`,
   `polymarket-wallet-grades`, `polymarket-sharp-money`, or
   `polymarket-market-research`.
3. https://0xinsider.com/api/v1/openapi.json for the typed contract.

## Rules for generated integration code

- **Use the sandbox first.** `https://0xinsider.com/sandbox` answers every
  documented operation with no credential. Shape the request there before you
  spend live quota. `?sandbox_status=<code>` exercises the error path.
- **Read credentials from the environment.** Never write an API key into a
  config file, a committed script, or an example. Use `OXINSIDER_API_KEY`.
- **Generate types from the OpenAPI spec.** Do not hand-write response
  interfaces; the spec is the source of truth and it moves.
- **Prefer the official SDKs** over a hand-rolled HTTP client: `0xinsider` on
  PyPI (import `oxinsider`), `github.com/0xinsider/0xinsider-go`, the Rust crate
  `oxinsider` (github.com/0xinsider/0xinsider-rust), `@0xinsider/sdk` for
  Node.js and TypeScript (github.com/0xinsider/0xinsider-node), and
  `@0xinsider/mcp` on npm for the CLI and MCP server.
- **Batch instead of looping.** Batch reads accept 25 items. A loop over 25
  addresses burns a quarter of the per-minute budget to fetch what one call
  returns.
- **Branch on `error.code`, never on the message.** Errors are typed JSON with
  a stable code and an `error.reason` that separates causes sharing a status.
- **Honor the rate-limit headers.** 100 requests per minute per API-key user.
  Read the headers; do not count requests client side. Honor `Retry-After`.
- **Send `Idempotency-Key` on writes.** Every retry of a POST needs the same
  key, or a timeout becomes a duplicate.
- **Follow cursors to completion.** Feeds and timelines are paged and move
  while you read. Never page by offset, and never total a partial page.

## Rules for presenting the data

These are correctness requirements, not style preferences. An agent that gets
them wrong produces a confident, wrong number.

- **An absent field is not a zero.** Unavailable, partial, or stale provider
  data marks the edge of the evidence. Serve a truthful unavailable state.
- **`pnl.realized` is omitted when no native accounting snapshot matches.**
  Raw total P&L is never a substitute for it.
- **Significance (0.0-1.0) ranks attention, not outcomes.** It is a separate
  quantity from Insider Radar's 0-100 review score. Neither is a
  probability, a forecast, or advice.
- **A zero-flow YES tie-break is not conviction.** `BUY YES` and `SELL NO` add
  exposure; `BUY NO` and `SELL YES` subtract it.
- **An ungraded wallet is uncovered, not unskilled.**
- **Keep full numeric precision until the final render.**
- **"Profitable wallets"** names the sharp cohort; **"tracked wallet"** means
  all-grades coverage. They are not interchangeable.
- **Every polymarket.com link carries `?r=0xinsidercom`.**

## Conventions in this repository

- No emojis, in code, docs, commits, or issues. Use plain-text markers.
- Every skill needs YAML frontmatter with `name` and a `description` that names
  its trigger phrases, so a dispatcher can route to it without reading the body.
- Keep `plugin.json` valid against
  `https://agent-plugins.org/schemas/1.0.0/plugin.schema.json`.

## Official resources

| Resource | URL |
| --- | --- |
| Developer portal | https://0xinsider.com/developers |
| API documentation | https://docs.0xinsider.com |
| OpenAPI 3.1 spec | https://0xinsider.com/api/v1/openapi.json |
| Authentication | https://0xinsider.com/auth.md |
| Versioning policy | https://0xinsider.com/api-versioning.md |
| Agent instructions | https://0xinsider.com/agents.md |
| Machine-readable index | https://0xinsider.com/llms.txt |
| Remote MCP endpoint | https://0xinsider.com/mcp |
| MCP server card | https://0xinsider.com/.well-known/mcp |
| Sandbox | https://0xinsider.com/sandbox/api/v1/leaderboard |
| Python SDK | https://github.com/0xinsider/0xinsider-python |
| Go SDK | https://github.com/0xinsider/0xinsider-go |
| Node.js and TypeScript SDK | https://github.com/0xinsider/0xinsider-node |
| Rust SDK | https://github.com/0xinsider/0xinsider-rust |
| CLI and MCP package | https://www.npmjs.com/package/@0xinsider/mcp |
| Homebrew tap | https://github.com/0xinsider/homebrew-tap |
| Research data | https://github.com/0xinsider/research |
