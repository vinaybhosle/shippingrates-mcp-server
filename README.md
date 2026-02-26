# ⚓ FreightGate MCP Server

> Container shipping intelligence for AI agents — demurrage & detention, local charges, inland haulage, CFS tariffs. Pay-per-request with USDC via x402.

[![MCP](https://img.shields.io/badge/MCP-Compatible-blue)](https://modelcontextprotocol.io)
[![x402](https://img.shields.io/badge/x402-USDC%20Payments-green)](https://x402.org)
[![Networks](https://img.shields.io/badge/Networks-Base%20%2B%20Solana-purple)]()
[![Version](https://img.shields.io/badge/version-2.1.0-orange)]()
[![Status](https://img.shields.io/badge/status-Production-brightgreen)]()

## Quick Start

Add FreightGate to Claude Desktop, Cursor, or any MCP-compatible client:

```json
{
  "mcpServers": {
    "freightgate": {
      "url": "https://mcp.shippingrates.org/mcp"
    }
  }
}
```

That's it. 9 tools auto-discovered. 3 free + 6 paid via x402.

## What's New in v2.1.0

- **Dual-chain payments** — Base Mainnet (EVM) + Solana Mainnet
- **Tiered pricing** — $0.03 to $0.25 per call based on data complexity
- **PayAI facilitator** — Free, no API key required for payment processing
- **23/23 endpoints verified** — Full production health check passed
- **2,547+ records** across 14 database tables
- **Server card** at `/.well-known/mcp/server-card.json` for auto-discovery
- **Live analytics** dashboard with real-time payment tracking
- **Listed on** Official MCP Registry, Smithery, Glama, mcp.so, x402scan

## Tools

### Paid (USDC via x402)

| Tool | Price | Description |
|------|-------|-------------|
| `dd_calculate` | $0.10 | Calculate demurrage & detention costs — free days, per-diem rates, slab breakdown, total cost |
| `dd_compare` | $0.25 | Compare D&D costs across all 6 shipping lines in one call. **No competitor offers this.** |
| `local_charges` | $0.05 | Port local charges — THC, documentation fees, BL fees, seal fees |
| `inland_search` | $0.03 | Search inland transport routes by origin/destination |
| `cfs_tariffs` | $0.05 | Container Freight Station handling tariffs by cargo type |
| `inland_haulage` | $0.05 | Inland haulage rates — trucking/rail from port to destination |

### Free

| Tool | Description |
|------|-------------|
| `list_lines` | List all 6 shipping lines with per-country record counts |
| `get_stats` | Database statistics — records, coverage, pricing, last update |
| `health_check` | Server status, version, network configuration |

## Payment

FreightGate uses the [x402 protocol](https://x402.org) — the internet's native payment standard for AI agents.

- **Currency:** USDC (stablecoin, always $1.00)
- **Networks:** Base Mainnet (EVM) + Solana Mainnet
- **Facilitator:** [PayAI](https://payai.network) — free, no API key needed
- **Settlement:** Instant (~200ms)
- **No API keys, no accounts, no subscriptions**

Your agent sends a request → gets HTTP 402 with payment terms → signs USDC payment → retries with proof → gets data. All handled automatically by x402 client libraries.

### Supported x402 Client Libraries

- **TypeScript:** `@x402/fetch` or `@x402/axios`
- **Python:** `x402` package (httpx / requests)
- **Go:** `x402` Go module

### Payment Wallets

| Network | Address |
|---------|---------|
| Base (EVM) | `0x8c9e0882b4c6e6568fe76F16D59F7E080465E5C8` |
| Solana | `Gcv56hKWuEGmXheBpJxAwQxvX6QAMimZzzVrHaVCbNWE` |

## Data Coverage

| Metric | Value |
|--------|-------|
| **Shipping Lines** | 6 — Maersk, MSC, CMA-CGM, Hapag-Lloyd, ONE, COSCO |
| **Records** | 2,547+ verified tariff entries across 14 tables |
| **Countries** | 9 — IN, AE, SG, CN, MY, LK, PK, BD, TH |
| **Container Types** | 20DV, 40DV, 40HC, 20RF, 40RF, 20OT, 40OT, 20FR, 40FR |
| **Data Quality** | 100% manually verified from official carrier PDFs |
| **Data Tables** | dd_tariffs, local_charges, inland_haulage, cfs_tariffs, ports, and more |

## API Access

For direct API integration (without MCP):

| Resource | URL |
|----------|-----|
| API Base | `https://api.shippingrates.org` |
| Full Docs | `https://api.shippingrates.org/llms-full.txt` |
| LLMs.txt | `https://api.shippingrates.org/llms.txt` |
| x402 Discovery | `https://api.shippingrates.org/.well-known/x402.json` |
| MCP Server | `https://mcp.shippingrates.org/mcp` |
| MCP Server Card | `https://mcp.shippingrates.org/.well-known/mcp/server-card.json` |
| Health Check | `https://api.shippingrates.org/health` |

### Example: D&D Calculate

```bash
# Free endpoint — no payment needed
curl https://api.shippingrates.org/api/stats

# Paid endpoint — returns HTTP 402 with payment instructions
curl -X POST https://api.shippingrates.org/api/dd/calculate \
  -H "Content-Type: application/json" \
  -d '{"line": "maersk", "country": "IN", "container_type": "40HC", "days": 14}'
```

### Example Response (D&D Calculate)

```json
{
  "line": "maersk",
  "country": "IN",
  "container_type": "40HC",
  "days": 14,
  "free_days": 4,
  "currency": "USD",
  "slabs": [
    { "from": 5, "to": 7, "rate_per_day": 7.5, "days": 3, "cost": 22.5 },
    { "from": 8, "to": 14, "rate_per_day": 15, "days": 7, "cost": 105 }
  ],
  "total_cost": 127.5
}
```

### Example Response (D&D Compare)

```json
{
  "country": "IN",
  "container_type": "40HC",
  "days": 14,
  "comparisons": [
    { "line": "maersk", "total_cost": 127.5, "free_days": 4, "currency": "USD" },
    { "line": "msc", "total_cost": 150.0, "free_days": 3, "currency": "USD" },
    { "line": "cosco", "total_cost": 95.0, "free_days": 7, "currency": "USD" }
  ],
  "cheapest": "cosco",
  "savings_vs_most_expensive": 55.0
}
```

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                   AI Agent / Client                  │
│            (Claude, Cursor, ChatGPT, etc.)           │
└──────────────┬──────────────────────┬───────────────┘
               │ MCP                  │ HTTP + x402
               ▼                      ▼
┌──────────────────────┐  ┌──────────────────────────┐
│  MCP Server v1.1.0   │  │    API Server v2.1.0     │
│  mcp.shippingrates   │──│  api.shippingrates.org   │
│  .org:3480           │  │  :3402                   │
└──────────────────────┘  └────────────┬─────────────┘
                                       │
                          ┌────────────▼─────────────┐
                          │  Shipping Intelligence DB │
                          │  SQLite — 14 tables       │
                          │  2,547+ verified records   │
                          └──────────────────────────┘
```

- **MCP Server** — TypeScript, `@modelcontextprotocol/sdk`, Streamable HTTP transport
- **API Server** — TypeScript, Express.js, x402 v2 SDK with PayAI facilitator
- **Database** — SQLite with data from official carrier tariff PDFs
- **Infrastructure** — PM2 process manager, Cloudflare Tunnel, reboot-proof

## Use Cases

- **Freight forwarders** estimating penalty costs before booking
- **AI agents** optimizing container pickup schedules to minimize D&D charges
- **Logistics companies** comparing carriers to find the cheapest option
- **Trade finance** assessing cost risk on import operations
- **Supply chain platforms** integrating real-time D&D data

## Discovery & Directories

FreightGate is listed on:

| Directory | Status |
|-----------|--------|
| [Official MCP Registry](https://registry.modelcontextprotocol.io) | ✅ Live — `io.github.vinaybhosle/freightgate-mcp-server` |
| [Smithery](https://smithery.ai) | ✅ Submitted |
| [Glama](https://glama.ai/mcp/servers) | ✅ Submitted |
| [mcp.so](https://mcp.so) | ✅ Submitted |
| [x402scan](https://x402scan.com) | ✅ Indexed — 6 resources auto-discovered |

## About

Built by [SmartBiz AI](https://shippingrates.org) — AI solutions for commercial shipping, based in Mumbai, India.

FreightGate is the world's first x402-native shipping data API, purpose-built for AI agents and autonomous commerce.

## Links

- 🌐 [Website](https://shippingrates.org)
- 📖 [Full API Documentation](https://api.shippingrates.org/llms-full.txt)
- 🔍 [x402 Discovery](https://api.shippingrates.org/.well-known/x402.json)
- ⚡ [x402 Protocol](https://x402.org)
- 🤖 [MCP Specification](https://modelcontextprotocol.io)
- 📊 [API Stats](https://api.shippingrates.org/api/stats)

## License

This repository contains documentation and connection configuration only. The FreightGate API and MCP server are commercial services operated by SmartBiz AI.

Usage of the API is governed by pay-per-request pricing via the x402 protocol.
