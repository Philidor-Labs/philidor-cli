<div align="center">

# Philidor CLI

### DeFi vault intelligence from your terminal

[![npm](https://img.shields.io/npm/v/@philidorlabs/cli.svg)](https://www.npmjs.com/package/@philidorlabs/cli)
[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue.svg)](https://www.typescriptlang.org)
[![Node](https://img.shields.io/badge/node-%3E%3D22-green.svg)](https://nodejs.org)

Risk scores, yield comparison, portfolio analysis, and oracle monitoring across 700+ DeFi vaults on Morpho, Aave, Yearn, Beefy, Spark, and more.

**No API key required.**

[Install](#install) &bull; [Commands](#commands) &bull; [Screening](#vault-screening) &bull; [Risk Framework](#risk-scoring) &bull; [Agent Integration](#agent-integration)

</div>

---

## Why Philidor?

Most DeFi tools are browser-only dashboards. Philidor gives you **institutional-grade risk intelligence in your terminal** &mdash; scriptable, pipeable, and purpose-built for both humans and AI agents.

| Feature | Philidor CLI | DeFi Dashboards | Generic APIs |
|---|:---:|:---:|:---:|
| Vault risk scores (0&ndash;10) | :white_check_mark: | Partial | :x: |
| Risk vector decomposition | :white_check_mark: | :x: | :x: |
| Side-by-side vault comparison | :white_check_mark: | Partial | :x: |
| Portfolio risk analysis | :white_check_mark: | :x: | :x: |
| Curator intelligence | :white_check_mark: | :x: | :x: |
| Oracle freshness monitoring | :white_check_mark: | :x: | :x: |
| Vault screening presets | :white_check_mark: | :x: | :x: |
| JSON / CSV / Table output | :white_check_mark: | :x: | JSON only |
| Agent sandboxing | :white_check_mark: | :x: | :x: |
| No API key needed | :white_check_mark: | :white_check_mark: | Varies |

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
philidor search "Gauntlet" --limit 20

# List and filter vaults
philidor vaults --chain ethereum
philidor vaults --protocol morpho --risk-tier prime
philidor vaults --asset USDC --chain base --min-tvl 1000000
philidor vaults --stablecoin --audited --high-confidence
philidor vaults --sort apr_net:desc --limit 10
```

### Vault Screening

Screen vaults with named presets or custom multi-criteria filters. Presets provide opinionated defaults that CLI flags can override.

```bash
# Named presets
philidor screen conservative              # Audited stablecoin vaults, risk score >= 7
philidor screen balanced                   # Audited vaults, risk score >= 5, sorted by APR
philidor screen aggressive                 # High-yield vaults, APR >= 5%
philidor screen stablecoin-yield           # Stablecoins sorted by yield
philidor screen blue-chip                  # TVL > $10M, audited, risk score >= 6

# Override preset defaults
philidor screen conservative --min-tvl 1000000
philidor screen balanced --chain ethereum --asset WETH

# Custom screening with range filters
philidor screen --min-apr 5 --min-score 7
philidor screen --chain base --curator gauntlet
philidor screen --stablecoin --single-exposure --audited
```

### Safest Vaults

```bash
philidor safest
philidor safest --asset USDC --chain ethereum --min-tvl 1000000
```

### Vault Detail

```bash
# By vault ID or network + address
philidor vault <vault-id>
philidor vault ethereum 0x98c23e9d8f34fefb1b7bd6a91b7ff122f4e16f5c

# Sub-resources
philidor vault ethereum 0x1234... --events       # Event history
philidor vault ethereum 0x1234... --markets      # Morpho market allocations
philidor vault ethereum 0x1234... --strategies   # Yearn strategies
philidor vault ethereum 0x1234... --rewards      # Reward breakdown
philidor vault ethereum 0x1234... --strategy     # Beefy strategy detail
```

### Portfolio Analysis

```bash
philidor portfolio 0xWalletAddress
philidor portfolio 0xWalletAddress --chain base
```

Returns vault details, balance in USD, APR, risk score, and risk tier for each position, plus aggregates: total value, weighted risk, position count, risk distribution.

### Comparison & Risk

```bash
# Side-by-side comparison (2-5 vaults)
philidor compare <vault-id-1> <vault-id-2> <vault-id-3>

# Risk vector breakdown
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

## Output Formats

All commands support three output modes:

```bash
philidor vaults --table      # Human-readable table (default in TTY)
philidor vaults --json       # Structured JSON (best for scripts and agents)
philidor vaults --csv        # CSV for spreadsheets and data pipelines
```

When piped (non-TTY), JSON is the default.

### Field Projection

```bash
# Select specific fields
philidor vaults --json --select name,apr_net,tvl_usd --results-only

# Strip pagination wrappers
philidor vaults --json --results-only --limit 5

# Works with table and CSV too
philidor vaults --table --select Name,APR,TVL
```

### Configuration

Persistent defaults via `~/.config/philidor/config.json`:

```json
{
  "apiUrl": "https://api.philidor.io",
  "defaultFormat": "table"
}
```

CLI flags and the `PHILIDOR_API_URL` environment variable always override config file values.

---

## Examples

### Find the safest USDC vaults on Base

```bash
philidor vaults --asset USDC --chain base --risk-tier prime --sort tvl_usd:desc --json
```

### Compare top Morpho vaults by yield

```bash
philidor vaults --protocol morpho --sort apr_net:desc --limit 3 --json
philidor compare <id-1> <id-2> <id-3> --json
```

### Audit a vault before depositing

```bash
philidor vault ethereum 0x98c23e...
philidor risk breakdown ethereum 0x98c23e...
philidor risk incidents --json
```

### Portfolio risk check

```bash
philidor portfolio 0xYourAddress --json | jq '.positions[] | select(.risk_tier == "Edge")'
```

### Export to spreadsheet

```bash
philidor vaults --protocol aave-v3 --csv > aave-vaults.csv
```

---

## Risk Scoring

Philidor uses the **Vector Risk Framework v4.1** to decompose vault risk into three measurable vectors:

```
Final Score = 40% Asset + 40% Platform + 20% Governance
```

### Asset Composition (40%)

Quality of underlying collateral. Blue-chip assets (ETH, USDC) score highest. Factors include oracle reliability, liquidity depth, and peg stability.

### Platform Code (40%)

Code maturity measured by:

- **Lindy Score** &mdash; time-based safety (>2 years &asymp; 9/10)
- **Audit Density** &mdash; number and quality of audits
- **Dependency Risk** &mdash; multiplicative penalties for risky dependencies
- **Incident Penalty** &mdash; caps score after security incidents

### Governance (20%)

Exit window for users:

| Control | Score |
|---|---|
| Immutable contract | 10/10 |
| 7+ day timelock | 9/10 |
| No timelock | 1/10 |

### Risk Tiers

| Tier | Score | Meaning |
|---|---|---|
| **Prime** | 8.0&ndash;10.0 | Institutional-grade &mdash; mature code, multiple audits, safe governance |
| **Core** | 5.0&ndash;7.9 | Moderate safety &mdash; audited but newer or flexible governance |
| **Edge** | 0.0&ndash;4.9 | Higher risk &mdash; requires careful due diligence |

---

## Agent Integration

The CLI is designed to work as a tool backend for AI agents and automated workflows.

### Agent Skills

Install the Philidor skill into your coding agent via [skills.sh](https://skills.sh):

```bash
npx skills add philidor-labs/philidor-cli
```

This gives your agent full knowledge of all commands, workflows, and best practices.

### Agent-Optimised Features

```bash
# Structured JSON output for parsing
philidor vaults --asset USDC --risk-tier prime --json

# Strip pagination wrappers
philidor vaults --json --results-only

# Project specific fields
philidor vaults --json --select name,apr_net,risk_tier

# Command sandboxing — restrict which commands an agent can invoke
philidor --enable-commands vaults,safest,compare vaults --json

# Full command tree as JSON for agent discovery
philidor schema

# Shell completion
philidor completion bash >> ~/.bashrc
```

### Also Available

| Interface | Description | Link |
|---|---|---|
| **MCP Server** | Native integration with Claude, Cursor, Windsurf &mdash; no CLI needed | [philidor-mcp](https://github.com/Philidor-Labs/philidor-mcp) |
| **OpenClaw Skill** | Skill definition for the OpenClaw agent platform | [npm](https://www.npmjs.com/package/@philidorlabs/openclaw-skill) |

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

700+ vaults tracked with continuous risk scoring.

---

## API

The CLI connects to the [Philidor Public API](https://api.philidor.io/v1/docs) at `https://api.philidor.io`. No authentication required.

Override the endpoint with `--api-url <url>` or the `PHILIDOR_API_URL` environment variable.

---

## Links

- [Philidor Analytics](https://app.philidor.io) &mdash; explore vaults and risk scores
- [Philidor MCP Server](https://github.com/Philidor-Labs/philidor-mcp) &mdash; AI agent integration via MCP
- [API Documentation](https://api.philidor.io/v1/docs) &mdash; OpenAPI/Swagger docs
- [Risk Methodology](https://app.philidor.io/methodology) &mdash; how scores are calculated
- [npm](https://www.npmjs.com/package/@philidorlabs/cli) &mdash; package registry
- [Twitter](https://twitter.com/philidorlabs) &mdash; updates and announcements

## License

[MIT](LICENSE)
