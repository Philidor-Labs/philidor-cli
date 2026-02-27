<div align="center">

# Philidor CLI

### DeFi vault intelligence from your terminal

[![npm](https://img.shields.io/npm/v/@philidorlabs/cli.svg)](https://www.npmjs.com/package/@philidorlabs/cli)
[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue.svg)](https://www.typescriptlang.org)
[![Node](https://img.shields.io/badge/node-%3E%3D22-green.svg)](https://nodejs.org)

Risk scores, yield comparison, portfolio analysis, and oracle monitoring across 700+ vaults on Morpho, Aave, Yearn, Beefy, Spark, and more.

**No API key required.**

[Install](#install) &bull; [Commands](#commands) &bull; [Examples](#examples) &bull; [Risk Framework](#risk-scoring) &bull; [Output Formats](#output-formats)

</div>

---

## Why Philidor CLI?

Most DeFi tools are browser-only. Philidor gives you **risk intelligence in your terminal** &mdash; scriptable, pipeable, and agent-friendly.

| Feature | Philidor CLI | DeFi Dashboards | Generic APIs |
|---|:---:|:---:|:---:|
| Vault risk scores (0-10) | :white_check_mark: | Partial | :x: |
| Risk vector decomposition | :white_check_mark: | :x: | :x: |
| Side-by-side vault comparison | :white_check_mark: | Partial | :x: |
| Portfolio risk analysis | :white_check_mark: | :x: | :x: |
| Curator intelligence | :white_check_mark: | :x: | :x: |
| Oracle freshness monitoring | :white_check_mark: | :x: | :x: |
| JSON / CSV / Table output | :white_check_mark: | :x: | JSON only |
| No API key needed | :white_check_mark: | :white_check_mark: | Varies |
| Scriptable & pipeable | :white_check_mark: | :x: | :white_check_mark: |

---

## Install

```bash
npm install -g @philidorlabs/cli
```

Verify:

```bash
philidor --version
philidor stats
```

---

## Commands

### Discovery & Search

```bash
# Platform overview — TVL, vault counts, risk distribution
philidor stats

# Search vaults by name, symbol, asset, or protocol
philidor search "USDC"
philidor search "Aave"
philidor search "Gauntlet" --limit 20

# List and filter vaults
philidor vaults
philidor vaults --chain ethereum
philidor vaults --protocol morpho --risk-tier prime
philidor vaults --asset USDC --chain base --min-tvl 1000000
philidor vaults --sort apr_net:desc --limit 10
```

### Vault Detail

```bash
# By vault ID
philidor vault <vault-id>

# By network + address
philidor vault ethereum 0x98c23e9d8f34fefb1b7bd6a91b7ff122f4e16f5c

# Sub-resources (require network + address)
philidor vault ethereum 0x1234... --events       # Event history
philidor vault ethereum 0x1234... --markets      # Morpho market allocations
philidor vault ethereum 0x1234... --strategies   # Yearn strategies
philidor vault ethereum 0x1234... --rewards      # Reward breakdown
```

### Portfolio Analysis

```bash
# All positions across all chains
philidor portfolio 0xYourWalletAddress

# Filter by chain
philidor portfolio 0xYourWalletAddress --chain base
```

Returns: vault details, balance in USD, APR, risk score, risk tier for each position. Includes aggregates: total value, weighted risk, position count, risk distribution.

### Comparison & Risk

```bash
# Side-by-side comparison (2-5 vaults)
philidor compare <vault-id-1> <vault-id-2> <vault-id-3>

# Risk vector breakdown (by ID or network + address)
philidor risk breakdown <vault-id>
philidor risk breakdown ethereum 0x1234...

# Risk methodology
philidor risk explain

# Vaults with recent critical incidents
philidor risk incidents
```

### Reference Data

```bash
philidor protocols                  # All protocols with vault counts and TVL
philidor protocol morpho            # Protocol detail — audits, incidents, chains
philidor curators                   # Curators with TVL and vault counts
philidor curator gauntlet           # Curator detail — managed vaults, performance
philidor chains                     # Supported chains
philidor assets                     # Supported assets
philidor oracles freshness          # Oracle feed health and deviation data
```

---

## Examples

### Find the safest USDC vaults on Base

```bash
philidor vaults --asset USDC --chain base --risk-tier prime --sort tvl_usd:desc --json
```

### Compare top Morpho vaults by yield

```bash
# Get the top 3
philidor vaults --protocol morpho --sort apr_net:desc --limit 3 --json

# Compare them
philidor compare <id-1> <id-2> <id-3> --json
```

### Audit a vault before depositing

```bash
# Full vault detail
philidor vault ethereum 0x98c23e9d8f34fefb1b7bd6a91b7ff122f4e16f5c

# Deep risk breakdown
philidor risk breakdown ethereum 0x98c23e9d8f34fefb1b7bd6a91b7ff122f4e16f5c

# Check for incidents
philidor risk incidents --json
```

### Portfolio risk check

```bash
philidor portfolio 0xYourAddress --json | jq '.positions[] | select(.risk_tier == "Edge")'
```

### Pipe to spreadsheet

```bash
philidor vaults --protocol aave-v3 --csv > aave-vaults.csv
```

### Script: monitor vault risk daily

```bash
#!/bin/bash
VAULT="aave-1-0x98c23e9d8f34fefb1b7bd6a91b7ff122f4e16f5c"
SCORE=$(philidor vault $VAULT --json | jq -r '.vault.total_score')
echo "$(date): Risk score = $SCORE"
if (( $(echo "$SCORE < 5" | bc -l) )); then
  echo "WARNING: Vault dropped to Edge tier!"
fi
```

---

## Output Formats

All commands support three output formats:

```bash
philidor vaults --table      # Human-readable table (default in TTY)
philidor vaults --json       # Structured JSON (best for scripts & agents)
philidor vaults --csv        # CSV for spreadsheets and data pipelines
```

When piped (non-TTY), JSON is the default.

### Global options

```bash
--api-url <url>              # Override API base URL
                             # Also respects PHILIDOR_API_URL env var
```

---

## Risk Scoring

Philidor uses the **Vector Risk Framework** to decompose vault risk into three measurable vectors:

```
Final Score = 40% Asset + 40% Platform + 20% Governance
```

### Asset Composition (40%)
Quality of underlying collateral. Blue-chip assets (ETH, USDC) score 10/10. Less liquid or exotic collateral scores lower.

### Platform Code (40%)
Code maturity measured by:
- **Lindy Score** &mdash; time-based safety (>2 years = ~9/10)
- **Audit Density** &mdash; number and quality of audits
- **Dependency Risk** &mdash; multiplicative penalties for risky dependencies
- **Incident Penalty** &mdash; caps score after security incidents

### Governance (20%)
Exit window for users:
- Immutable contract: 10/10
- 7+ day timelock: 9/10
- No timelock: 1/10

### Risk Tiers

| Tier | Score | Meaning |
|---|---|---|
| **Prime** | 8.0 - 10.0 | Institutional-grade &mdash; mature code, multiple audits, safe governance |
| **Core** | 5.0 - 7.9 | Moderate safety &mdash; audited but newer or flexible governance |
| **Edge** | 0.0 - 4.9 | Higher risk &mdash; requires careful due diligence |

---

## For AI Agents

The CLI is designed to work as a tool backend for AI agents. Use `--json` for structured output:

```bash
# Agent workflow: find → compare → deep-dive
philidor vaults --asset USDC --risk-tier prime --json
philidor compare <id-1> <id-2> --json
philidor risk breakdown <chosen-id> --json
philidor risk incidents --json
```

See also: **[@philidorlabs/openclaw-skill](https://www.npmjs.com/package/@philidorlabs/openclaw-skill)** for the OpenClaw agent skill definition, and **[Philidor MCP Server](https://github.com/Philidor-Labs/philidor-mcp)** for native MCP integration with Claude, Cursor, and Windsurf.

---

## Supported Protocols

| Protocol | Chains |
|---|---|
| **Aave v3** | Ethereum, Base, Arbitrum, Polygon, Optimism, Avalanche |
| **Morpho** | Ethereum, Base |
| **Yearn v3** | Ethereum, Polygon, Arbitrum, Base, Optimism |
| **Beefy** | Multi-chain (12+) |
| **Spark** | Ethereum |
| **Compound** | Ethereum |
| **Uniswap** | Ethereum, Base, Arbitrum, Polygon, Optimism |

700+ vaults tracked with real-time risk scoring.

---

## API

The CLI connects to the public Philidor API at `https://api.philidor.io`. No authentication required.

- [API Documentation](https://api.philidor.io/v1/docs) &mdash; OpenAPI/Swagger
- Override endpoint: `--api-url <url>` or `PHILIDOR_API_URL` env var

---

## Links

- [Philidor Analytics](https://app.philidor.io) &mdash; explore vaults and risk scores
- [Philidor MCP Server](https://github.com/Philidor-Labs/philidor-mcp) &mdash; AI agent integration via MCP
- [API Documentation](https://api.philidor.io/v1/docs) &mdash; OpenAPI/Swagger docs
- [Risk Methodology](https://app.philidor.io/methodology) &mdash; how scores are calculated
- [npm: @philidorlabs/cli](https://www.npmjs.com/package/@philidorlabs/cli) &mdash; npm package
- [npm: @philidorlabs/openclaw-skill](https://www.npmjs.com/package/@philidorlabs/openclaw-skill) &mdash; OpenClaw skill
- [Twitter](https://twitter.com/philidorlabs) &mdash; updates and announcements

## License

[MIT](LICENSE)
