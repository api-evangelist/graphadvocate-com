---
name: Graphadvocate
description: Use when routing blockchain data queries to the right subgraph, scoring traders on Hyperliquid or Polymarket, analyzing x402 payment activity, discovering active x402 services, or calling Graph Advocate from agent-to-agent (A2A), MCP, or x402-aware clients. Reach for this skill when an agent needs to ask plain-English questions about onchain data and get back ready-to-run GraphQL queries, derived trader intelligence, or payment analytics.
metadata:
    mintlify-proj: graphadvocate
    version: "1.0"
---

# GraphAdvocate Skill

## Product summary

GraphAdvocate is a routing agent for onchain data that accepts plain-English questions and returns the right subgraph, a ready-to-run GraphQL query, and (when possible) the live answer. It routes 15,000+ subgraphs across 70+ networks and speaks A2A JSON-RPC, MCP SSE, and x402 payments. The primary endpoint is `https://graphadvocate.com/` with MCP at `https://graphadvocate.com/mcp`. Key features: free tier (3 routing calls/day for identified senders), paid routing at $0.01 USDC on Base via x402, trader intelligence endpoints for Hyperliquid and Polymarket ($0.01–$0.10), natural-language SQL over x402 settlements, and live Bazaar discovery. No API keys, no signup, no card — your agent's wallet is the billing relationship. Registered as ERC-8004 agent #734 on Arbitrum.

## When to use

**Routing queries:** When an agent needs to find the right subgraph for a blockchain data question (e.g., "Top Uniswap V3 pools on Base", "Aave V3 liquidation risk", "Wallet balance for vitalik.eth"). Use the free tier (`POST /`) for 3 calls/day per identified sender; use paid routing (`POST /route`) for unlimited calls at $0.01 each.

**Trader intelligence:** When scoring or screening traders on Hyperliquid (skill scores, liquidation risk, vault evaluation) or Polymarket (PnL, win rate, ghost-fill counterparty risk). These are derived signals, not raw data — use them for pre-trade risk assessment, vault evaluation, or market-sizing decisions.

**x402 analytics:** When analyzing payment activity on Base — top recipients, volume trends, facilitator share — via natural-language SQL (`POST /ask`, $0.05).

**Service discovery:** When finding which x402 services are actually paid and verified right now via the triple-join Bazaar (`/bazaar/active`).

**Agent-to-agent calls:** When another agent needs to discover GraphAdvocate's capabilities via the agent card (`/.well-known/agent-card.json`) or call it from MCP or A2A JSON-RPC.

## Quick reference

### Endpoints and pricing

| Endpoint | Method | Cost | Use for |
|---|---|---|---|
| `POST /` | A2A JSON-RPC | Free (3/day identified) | Routing queries, free tier |
| `POST /route` | REST | $0.01 USDC | Paid routing, unlimited |
| `POST /hyperliquid/score` | REST | $0.02 USDC | Trader skill score |
| `POST /hyperliquid/screen` | REST | $0.05 USDC | Top N traders on a coin |
| `POST /hyperliquid/vault` | REST | $0.10 USDC | Vault evaluation |
| `POST /polymarket/pnl-quick` | REST | $0.01 USDC | Fast wallet skill read |
| `POST /polymarket/screen` | REST | $0.02 USDC | Market holders + risk |
| `POST /ask` | REST | $0.05 USDC | NL-SQL over x402 settlements |
| `GET /bazaar/active` | REST | Free | Live active x402 services |
| `GET /.well-known/agent-card.json` | REST | Free | A2A discovery metadata |

### Free tier identification

Include wallet address or agent ID in request metadata to unlock 3 free routing calls/day:

```json
{
  "params": {
    "metadata": {
      "sender_id": "0x...",
      "from_agent_id": "42161:734"
    }
  }
}
```

Anonymous requests (no identity) pay from call 1.

### Response fields (routing)

- **`recommendation`** — service type (subgraph-registry, token-api, substreams, mcp-package)
- **`gql`** — production-ready GraphQL, paste as-is
- **`subgraph_id`** — deployment ID for direct gateway queries
- **`curl_example`** — complete working curl command
- **`execution_result`** — live answer (when available; no follow-up call needed)
- **`alternatives`** — ranked alternatives with reasons

### MCP packages (free, npx-installable)

| Package | Coverage | Install |
|---|---|---|
| `graph-aave-mcp` | Aave V2/V3/V4, 40+ tools | `npx graph-aave-mcp` |
| `graph-uniswap-mcp` | Uniswap V2/V3/V4, 8 tools, 6 chains | `npx -y graph-uniswap-mcp` |
| `graph-polymarket-mcp` | Polymarket, 35 tools, CLOB V2 attribution | `npx graph-polymarket-mcp` |
| `graph-lending-mcp` | Cross-protocol lending (Messari schema) | `npx graph-lending-mcp` |
| `graph-limitless-mcp` | Limitless prediction markets on Base | `npx graph-limitless-mcp` |

Most require `GRAPH_API_KEY` from thegraph.com/studio.

### x402 payment flow

1. First call to x402-gated endpoint returns `HTTP 402` with payment requirements
2. x402-aware client signs `transferWithAuthorization` (USDC on Base)
3. CDP facilitator verifies and settles on-chain
4. Handler processes request and returns result
5. Payment triggers CDP Bazaar indexing automatically

## Decision guidance

### When to use free tier vs. paid routing

| Scenario | Use | Reason |
|---|---|---|
| Exploring, prototyping, low-volume queries | `POST /` (free tier) | 3 calls/day per identified sender, no payment setup |
| Production agent, high-volume queries | `POST /route` (paid) | Unlimited calls at $0.01 each, x402 auto-retry |
| Inspecting routing without paying | `POST /` with plain-English question | Returns recommendation + exact curl for paid execution |

### When to use MCP vs. paid endpoints

| Scenario | Use | Reason |
|---|---|---|
| Exploring raw data (pools, markets, positions) | MCP package | Free, your own Graph API key, direct subgraph access |
| Derived answers (basis JOIN, liquidation risk, skill scores) | Paid endpoint | Synthesized once, no client-side parsing, ready-to-act signal |
| Cross-protocol comparison | `graph-lending-mcp` | Messari standardized schema, direct comparability |

### When to use Hyperliquid vs. Polymarket endpoints

| Scenario | Use | Reason |
|---|---|---|
| Perps trader evaluation, vault screening | `/hyperliquid/*` | Skill score (40% profitability, 40% risk, 20% efficiency), liquidation rate, funding burn |
| Prediction market wallet assessment | `/polymarket/pnl-quick` | Fast skill read ($0.01), win rate, realized PnL |
| Market-sizing, counterparty risk | `/polymarket/screen` + `/polymarket/risk` | Ghost-fill detection (smart-contract vs. EOA), holder concentration |

## Workflow

### Routing a blockchain data query

1. **Identify the question** — e.g., "Top Uniswap V3 pools on Base by volume"
2. **Check free tier eligibility** — include `sender_id` or `from_agent_id` in metadata to unlock 3 free calls/day
3. **POST to `/` (free) or `/route` (paid)** — send plain-English question in `params.message.parts[].text` (A2A) or `{"request": "..."}` (REST)
4. **Parse response** — extract `gql`, `subgraph_id`, or `execution_result`
5. **Use the answer** — if `execution_result` is present, you're done; otherwise fire `curl_example` against the Graph gateway with your own API key

### Scoring a Hyperliquid trader

1. **Collect wallet address** — e.g., `0x2fc3195efbf91ad90854bc3c02fe739895c23460`
2. **POST to `/hyperliquid/score`** — send `{"user": "0x..."}`
3. **Read the signal** — `skill_score` (0–100), `classification` (sharp/neutral/retail), `liquidation_rate_bps`, `profit_factor`
4. **Act on confidence** — `confidence` field shrinks scores for small samples (<100 trades returns `insufficient_data`)

### Screening a Polymarket market

1. **Identify market** — market slug or condition ID
2. **POST to `/polymarket/screen`** — send `{"market": "..."}`
3. **Parse holders** — top N holders with `skill_score` and `ghost_fill_risk_score`
4. **Assess counterparty** — for any holder, POST to `/polymarket/risk` to detect wallet type (EOA vs. smart-contract)

### Querying x402 settlement activity

1. **Formulate question** — e.g., "Top 10 x402 recipients by USDC volume in the last 30 days"
2. **POST to `/ask`** — send `{"question": "..."}`
3. **Verify the path** — check `sql_trace` array to confirm the SQL that produced the answer
4. **Act on the answer** — use the derived signal; keep `sql_trace` for audit

## Common gotchas

- **Anonymous senders pay from call 1** — always include `sender_id` or `from_agent_id` in metadata to unlock free tier. No IP-fallback free tier exists.
- **`execution_result` is optional** — if missing, you must fire the `curl_example` against the Graph gateway yourself with your own API key. The `gql` is always live-tested before being returned, so it will work.
- **Hyperliquid/Polymarket scores require sample size** — under 100 trades returns `insufficient_data` regardless of score. Don't trust a 50-trade wallet's skill score.
- **Confidence shrinkage is real** — `confidence` field on Hyperliquid scores is `log10(transactions) / 6`, so a 1000-trade wallet gets ~0.5 confidence. Use it to weight your decision.
- **Ghost-fill risk is Polymarket-specific** — only relevant for newer API-user smart-contract wallets (ERC-1967 proxies). EOAs have zero ghost-fill risk.
- **x402 payments settle on Base only** — all payments are USDC on Base mainnet. No other networks supported yet.
- **MCP packages are free but need Graph API key** — most require `GRAPH_API_KEY` from thegraph.com/studio. Polymarket MCP needs no key.
- **Retired MCP packages 404** — `predictfun-mcp`, `substreams-search-mcp`, and `create-substreams-sink-sql` were unpublished. Use underlying REST APIs instead.
- **Bazaar discovery has cache warmup** — first request after cache expiry takes ~10s (8004scan multi-chain lookup). Subsequent requests return in <1s.
- **`/ask` has 60s timeout** — DuckDB queries over 132M rows can timeout. Tighten the question or retry; response includes `retry_after_seconds`.

## Verification checklist

Before submitting work with GraphAdvocate:

- [ ] **Free tier:** Included `sender_id` or `from_agent_id` in metadata if using `POST /`
- [ ] **Routing response:** Checked that `gql` is valid GraphQL and `subgraph_id` matches the recommendation
- [ ] **Execution result:** If `execution_result` is missing, confirmed you have a Graph API key and can fire the `curl_example`
- [ ] **Trader scores:** Verified `sample_size_trades` ≥ 100 before trusting the skill score; checked `confidence` field
- [ ] **Polymarket risk:** For ghost-fill assessment, confirmed `wallet_type` is `new_api_user_smart_account` (not EOA)
- [ ] **x402 query:** Reviewed `sql_trace` to confirm the SQL matches your question intent
- [ ] **Payment:** Confirmed x402 client is configured with Base mainnet and USDC
- [ ] **MCP packages:** If using MCP, confirmed `GRAPH_API_KEY` is set and the package installed without errors

## Resources

**Comprehensive page listing:** [https://docs.graphadvocate.com/llms.txt](https://docs.graphadvocate.com/llms.txt)

**Critical documentation:**
- [Quickstart](https://docs.graphadvocate.com/quickstart) — 30 seconds to first routing response
- [POST /route](https://docs.graphadvocate.com/route) — paid routing endpoint, response shape, pricing
- [Hyperliquid trader intelligence](https://docs.graphadvocate.com/hyperliquid) — skill scoring, vault evaluation, screening
- [Polymarket trader intelligence](https://docs.graphadvocate.com/polymarket) — PnL, ghost-fill risk, market screening
- [MCP packages](https://docs.graphadvocate.com/mcp-packages) — free protocol-specific servers (Aave, Uniswap, Polymarket, lending, Limitless)
- [Natural-language SQL over x402 settlements](https://docs.graphadvocate.com/ask) — `/ask` endpoint, DuckDB queries, sql_trace
- [Live Bazaar](https://docs.graphadvocate.com/bazaar-active) — active x402 services, triple-join discovery
- [Agent card](https://docs.graphadvocate.com/agent-card) — A2A discovery, ERC-8004 registration, registry lookups

---

> For additional documentation and navigation, see: https://docs.graphadvocate.com/llms.txt