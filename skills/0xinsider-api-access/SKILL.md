---
name: 0xinsider-api-access
description: Authenticate against the 0xinsider Developer API and its MCP server, pick between the sandbox and live hosts, and handle rate limits and errors. Use when connecting to 0xinsider, getting an API key, wiring the 0xinsider MCP server into an agent client, debugging a 401, 403 or 429 from api.0xinsider.com, or trying 0xinsider endpoints without an account.
license: MIT
metadata:
  publisher: 0xinsider
  homepage: https://0xinsider.com/developers
  docs: https://docs.0xinsider.com
  openapi: https://0xinsider.com/api/v1/openapi.json
---

# 0xinsider API access

0xinsider is Polymarket analytics for sports and esports in real-time. This
skill covers getting in: credentials, hosts, transports, and the failure modes
you will actually hit.

## Pick a host first

| Host | Credential | Data |
| --- | --- | --- |
| `https://0xinsider.com/sandbox` | none | Documented examples and deterministic schema samples. Nothing is stored. |
| `https://api.0xinsider.com` | live key, active Pro subscription | Live Polymarket data. |

Start in the sandbox. Every documented operation answers there with no
credential, so you can shape a request and assert on the response schema before
you spend a live quota unit:

```bash
curl -s https://0xinsider.com/sandbox/api/v1/leaderboard
```

Add `?sandbox_status=<code>` to receive one of the error responses that
operation documents. Use it to exercise your retry path:

```bash
curl -si 'https://0xinsider.com/sandbox/api/v1/leaderboard?sandbox_status=429'
```

The sandbox does not validate request bodies or parameters, does not simulate
streams or file downloads, and stores nothing.

## Get a credential with no account

One POST returns a sandbox key and the steps to live access:

```bash
curl -s -X POST https://api.0xinsider.com/api/v1/agents/register \
  -H 'Content-Type: application/json' \
  -d '{"name":"my-agent"}'
```

The key is prefixed `oxi_sk_test_`. It is a sandbox credential: sending it to
`api.0xinsider.com` returns `401 invalid_api_key` with `error.reason` set to
`sandbox_api_key`. That is the expected response, not a bug.

Live keys are prefixed `oxi_sk_live_` and require an active Pro subscription.

## Authenticate

Two mechanisms, both Bearer:

```bash
curl -s https://api.0xinsider.com/api/v1/leaderboard \
  -H "Authorization: Bearer $OXINSIDER_API_KEY"
```

- **API key** for scripts and server-to-server calls.
- **OAuth 2.1 authorization code with PKCE** for apps and MCP clients. The
  authorization-server metadata is at
  `https://api.0xinsider.com/.well-known/oauth-authorization-server`, and the
  protected-resource metadata follows RFC 9728.

Never hardcode a key. Read it from the environment at call time.

A `401` carries a `WWW-Authenticate` header naming the RFC 9728
`resource_metadata` URL, so a client can discover where to authenticate without
a hardcoded issuer. A `403 insufficient_scope` names the scope an OAuth token
is missing. Read the header rather than guessing.

## Discovery routes that need no credential

- `GET /api/v1` - live route index and capability document
- `GET /api/v1/health`
- `GET /api/v1/platforms` - platform capability matrix
- The MCP handshake: `initialize`, `ping`, `tools/list`

Data, export, webhook, usage, and MCP `tools/call` operations require a
credential and an active Pro subscription.

## MCP

The remote server speaks Streamable HTTP at `https://0xinsider.com/mcp`. Its
card, including transport and tool metadata, is at
`https://0xinsider.com/.well-known/mcp`.

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

A local stdio path ships in the CLI:

```bash
npx -y 0xinsider init
```

Install it permanently with `npm install --global 0xinsider` or
`brew install 0xinsider/tap/oxinsider`.

## Rate limits

100 requests per minute per API-key user. Batch reads accept 25 items per call
and reserve 2500 item units per minute.

Rate-limit headers are on every response. Read them instead of counting
requests yourself, and honor `Retry-After` on a `429`. Check consumption
without spending primary quota:

```bash
curl -s https://api.0xinsider.com/api/v1/usage \
  -H "Authorization: Bearer $OXINSIDER_API_KEY"
```

## Errors

Errors are JSON with a typed model: a stable `error.code`, a human `message`,
and an `error.reason` that distinguishes causes sharing one status. Branch on
`error.code`, never on the message string.

Write operations accept `Idempotency-Key`. Send one on every retry of a POST so
a timeout does not become a duplicate.

## Reading the data honestly

An unavailable, partial, or stale provider field marks the edge of the
evidence. Do not render it as a zero and do not describe it as a prediction.
When `pnl.realized` is absent, raw total P&L is not a substitute for it.

## References

- Docs portal: https://docs.0xinsider.com
- OpenAPI 3.1: https://0xinsider.com/api/v1/openapi.json
- Auth guide: https://0xinsider.com/auth.md
- Versioning policy: https://0xinsider.com/api-versioning.md
- Agent instructions: https://0xinsider.com/agents.md
