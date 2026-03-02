---
name: philidor-cli
description: >
  DeFi vault intelligence CLI: risk scores, yield comparison, portfolio analysis,
  and oracle monitoring across Morpho, Yearn, Aave, Beefy, and Spark.
  Use when the user wants to find safe DeFi vaults, compare yields,
  check vault risk scores, analyze a wallet's DeFi positions, search for
  yield opportunities, monitor oracle freshness, or get DeFi market overview stats.
  Covers Ethereum, Base, Arbitrum, Polygon, Optimism, and Avalanche.
---

# Philidor — DeFi Vault Intelligence

Institutional-grade risk scores, yield comparison, portfolio analysis, and oracle monitoring for DeFi vaults across five major protocols and six chains.

## When to Use This Skill

- User asks about DeFi vault risk, safety, or yield
- User wants to compare vaults or protocols
- User needs to analyze a wallet's DeFi positions
- User asks about DeFi market stats or oracle health
- User wants to screen vaults by criteria (risk tier, TVL, APR, chain)

## When NOT to Use This Skill

- User asks about token prices, swaps, or trading
- User asks about NFTs or non-DeFi topics
- User wants to execute transactions (this is read-only analytics)

## Prerequisites

Install the CLI:

```bash
npm install -g @philidorlabs/cli
```

## Quick Start

```bash
# Platform overview — total TVL, vault counts, risk distribution
philidor stats

# Search vaults by name, symbol, or asset
philidor search "USDC"

# Analyze all DeFi positions for a wallet address
philidor portfolio 0xYourWalletAddress
```

## Core Concepts

### Risk Tiers

Every vault receives a composite risk score from 0 to 10, grouped into three tiers:

| Tier        | Score Range | Meaning                                                                                        |
| ----------- | ----------- | ---------------------------------------------------------------------------------------------- |
| **Prime**   | 8.0 - 10.0  | Institutional-grade. Battle-tested protocols, blue-chip assets, strong governance controls.    |
| **Core**    | 5.0 - 7.9   | Solid fundamentals with some trade-offs in asset quality, audit coverage, or decentralisation. |
| **Edge**    | 0.0 - 4.9   | Higher risk. Newer protocols, exotic assets, weaker control structures, or recent incidents.   |
| **Unrated** | N/A         | Insufficient data to score. Treated as high-risk by default.                                   |

Scores are derived from three weighted vectors:

- **Asset Quality (40%)** — collateral quality, oracle reliability, liquidity depth, peg stability
- **Platform & Strategy (40%)** — smart contract maturity, audit coverage, TVL track record, incident history
- **Control & Governance (20%)** — timelock presence, multisig configuration, upgradeability, admin powers

### APR (Annual Percentage Rate)

APR values are stored as **decimals**: `0.05` means **5%**.

- **`apr_net`** — total APR including base yield plus any incentive rewards
- **`base_apr`** — native protocol yield only (lending rate, share price accrual), before rewards
- **Invariant**: `apr_net = base_apr + SUM(reward APRs)`

Reward types include `token_incentive` (MORPHO, ARB, SPK), `points`, `trading_fee` (LP fees), and `strategy` (Yearn sub-strategies).

### Protocols

| Protocol     | Description                                                                          |
| ------------ | ------------------------------------------------------------------------------------ |
| **Morpho**   | Optimised lending with curated vaults (Ethereum, Base)                               |
| **Yearn v3** | Automated yield strategies (Ethereum, Polygon, Arbitrum, Base, Optimism)             |
| **Aave v3**  | Blue-chip lending/borrowing (Ethereum, Polygon, Arbitrum, Avalanche, Optimism, Base) |
| **Beefy**    | Multi-chain yield optimiser (12+ chains)                                             |
| **Spark**    | MakerDAO lending markets (Ethereum)                                                  |

### Chains

Ethereum, Base, Arbitrum, Polygon, Optimism, and Avalanche.

## Commands

### Safest Vaults

```bash
# Find safest vaults — audited, high-confidence, sorted by risk score
philidor safest
philidor safest --asset USDC
philidor safest --chain ethereum
philidor safest --min-tvl 1000000
philidor safest --limit 5
```

### Vault Screening

Screen vaults with named presets or custom multi-criteria filters. Presets provide opinionated defaults that CLI flags can override.

```bash
# Named presets — opinionated screening profiles
philidor screen conservative              # Audited stablecoin vaults, risk score >= 7
philidor screen balanced                   # Audited vaults, risk score >= 5, sorted by APR
philidor screen aggressive                 # High-yield vaults, APR >= 5%
philidor screen stablecoin-yield           # Stablecoins sorted by yield
philidor screen blue-chip                  # TVL > $10M, audited, risk score >= 6

# Override preset defaults with flags
philidor screen conservative --min-tvl 1000000
philidor screen balanced --chain ethereum --asset WETH
philidor screen aggressive --protocol morpho

# Custom screening with range filters
philidor screen --min-apr 5 --min-score 7             # 5%+ APR with 7+ risk score
philidor screen --chain base --curator gauntlet        # Base chain, Gauntlet-curated
philidor screen --search morpho --lsd                  # LSD vaults matching "morpho"
philidor screen --stablecoin --single-exposure --audited

# All vaults flags plus new screen-only filters
philidor screen --min-apr 3 --max-apr 10 --min-score 5 --max-score 9
philidor screen --curator <name> --search <text> --lsd --single-exposure
```

### Discovery & Search

```bash
# List vaults with filters
philidor vaults
philidor vaults --chain ethereum
philidor vaults --protocol morpho
philidor vaults --risk-tier prime
philidor vaults --asset USDC
philidor vaults --min-tvl 1000000
philidor vaults --stablecoin --audited
philidor vaults --high-confidence             # Only vaults with high data confidence
philidor vaults --no-il                       # Exclude vaults with impermanent loss risk
philidor vaults --sort apr_net:desc --limit 10 --page 2

# Search vaults by name, protocol, asset, or address
philidor search "USDC"
philidor search "Aave"
philidor search "Gauntlet"
philidor search "USDC" --limit 20             # Control result count (default: 10)

# For filtered queries, use `vaults` with flags instead of search:
philidor vaults --asset USDC --chain base --risk-tier prime
philidor vaults --asset WETH --protocol morpho --sort apr_net:desc
```

### Vault Detail

```bash
# By vault ID
philidor vault <vault-id>

# By network slug + contract address
philidor vault ethereum 0x1234...

# Sub-resources (require network + address form)
philidor vault ethereum 0x1234... --events       # Event history (incidents, parameter changes)
philidor vault ethereum 0x1234... --markets      # Morpho market allocations
philidor vault ethereum 0x1234... --strategies   # Yearn strategies
philidor vault ethereum 0x1234... --rewards      # Reward breakdown (base + incentives)
philidor vault ethereum 0x1234... --strategy    # Beefy strategy — yield sources, risk flags, APY
```

### Portfolio

```bash
# All positions across all chains
philidor portfolio 0xWalletAddress

# Filter to a specific chain
philidor portfolio 0xWalletAddress --chain base
```

Returns: vault details, chain, balance in USD, APR, risk score, risk tier for each position. Includes aggregates: total value, weighted risk, position count, risk distribution.

### Comparison & Risk

```bash
# Side-by-side vault comparison (2-5 vault IDs)
philidor compare <vault-id-1> <vault-id-2> <vault-id-3>

# Detailed risk vector breakdown for a specific vault
philidor risk breakdown <vault-id>
philidor risk breakdown ethereum 0x1234...

# Explain the risk scoring methodology — tiers, vectors, weights
philidor risk explain

# Vaults with recent critical security incidents
philidor risk incidents
```

### Reference Data

```bash
# List all supported protocols with vault counts and TVL
philidor protocols

# Protocol detail — audit status, chain coverage, incident history
philidor protocol <id>

# List curators (Morpho vault managers)
philidor curators
philidor curators --sort tvl_total:desc --limit 10 --page 1

# Curator detail — managed vaults, TVL, performance
philidor curator <id>

# Platform overview — total TVL, vault counts, risk distribution
philidor stats

# Supported chains
philidor chains

# Supported assets
philidor assets

# Oracle feed freshness and deviation data
philidor oracles freshness
```

### Output Formats

All commands support three output formats:

```bash
philidor vaults --json       # Structured JSON (best for agents and scripting)
philidor vaults --table      # Human-readable table (default in TTY)
philidor vaults --csv        # CSV for spreadsheets and data pipelines
```

When output is piped (non-TTY), JSON is the default. Use `--json` explicitly in agent workflows for consistency.

Additional global flags:

```bash
--api-url <url>              # Override API base URL (default: https://api.philidor.io)
                             # Also respects PHILIDOR_API_URL environment variable
--select <fields>            # Comma-separated fields to include in output
--results-only               # Strip pagination metadata, return data array only
```

### Field Projection & Results-Only

Use `--select` to project specific fields from the JSON output, and `--results-only` to strip pagination wrappers:

```bash
# Get just names and APR for top vaults
philidor vaults --json --select name,apr_net,tvl_usd --results-only
# Returns: [{ "name": "...", "apr_net": 0.05, "tvl_usd": "..." }, ...]

# Works with table/csv too — filters columns by header name
philidor vaults --table --select Name,APR,TVL

# Combine for clean agent output
philidor vaults --json --results-only --select name,apr_net,risk_tier --limit 5
```

### Shell Completion

Generate shell completion scripts for tab-completion of commands and options:

```bash
philidor completion bash >> ~/.bashrc
philidor completion zsh > ~/.zshrc.d/_philidor
philidor completion fish > ~/.config/fish/completions/philidor.fish
```

### Configuration File

Persistent defaults via `~/.config/philidor/config.json`:

```json
{
  "apiUrl": "https://api.philidor.io",
  "defaultFormat": "table"
}
```

CLI flags always override config file values.

### Health Check

```bash
philidor health               # Check API connectivity, database status, vault count
philidor health --json        # Structured health response for monitoring scripts
```

### Agent Discovery

```bash
philidor schema               # Dump full command tree as JSON for agent introspection
```

### Agent Sandboxing

```bash
# Restrict which commands an agent can invoke
philidor --enable-commands vaults,safest,compare vaults --json
# Introspection commands (schema, version, completion, help) are always allowed
```

## Agent Workflows

### Workflow 1: Find the Best Vault for a User's Needs

```bash
# Step 1: Search for matching vaults
philidor search "stablecoin vault on base" --json

# Step 2: Compare the top candidates side-by-side
philidor compare <vault-id-1> <vault-id-2> <vault-id-3> --json

# Step 3: Deep-dive into the risk profile of the chosen vault
philidor risk breakdown <chosen-vault-id> --json

# Step 4: Check recent events for any red flags
philidor vault <network> <address> --events --json

# Decision: Recommend the vault with the best risk-adjusted yield.
# Flag any Edge-tier vaults or recent incidents to the user.
```

### Workflow 2: Compare Protocols for Yield Farming

```bash
# Step 1: Get protocol overview
philidor protocols --json

# Step 2: Get top vaults per protocol (can run in parallel)
philidor vaults --protocol morpho --sort apr_net:desc --limit 5 --json
philidor vaults --protocol aave-v3 --sort apr_net:desc --limit 5 --json
philidor vaults --protocol yearn-v3 --sort apr_net:desc --limit 5 --json

# Step 3: Compare the best vaults across protocols
philidor compare <best-morpho-id> <best-aave-id> <best-yearn-id> --json

# Step 4: Get protocol-level details
philidor protocol morpho --json

# Decision: Compare yield vs risk across protocols.
```

### Workflow 3: Analyze Portfolio Risk

```bash
# Step 1: Get all positions
philidor portfolio 0xWalletAddress --json

# Step 2: Identify risky positions (risk_tier = "Edge" or risk_score < 5)

# Step 3: Investigate each risky position
philidor risk breakdown <risky-vault-id> --json

# Step 4: Find safer alternatives for the same asset
philidor vaults --asset USDC --risk-tier prime --audited --json

# Decision: Present a portfolio risk summary.
# Highlight Edge-tier positions and suggest Prime alternatives.
```

### Workflow 4: Monitor Vault Safety

```bash
# Step 1: Check for active incidents across all protocols
philidor risk incidents --json

# Step 2: Check oracle health
philidor oracles freshness --json

# Step 3: Check event history for a specific vault
philidor vault <network> <address> --events --json

# Decision: Flag vaults with active incidents or stale oracle feeds.
```

## Interpreting Output

Key fields and their formats:

| Field            | Format          | Example                                   | Notes                                                            |
| ---------------- | --------------- | ----------------------------------------- | ---------------------------------------------------------------- |
| `apr_net`        | Decimal         | `0.0523`                                  | Total APR = 5.23%. Includes base yield + rewards.                |
| `base_apr`       | Decimal         | `0.0340`                                  | Base yield only = 3.40%. Native protocol rate before incentives. |
| `tvl_usd`        | Number (string) | `"12500000.50"`                           | Total value locked in USD. Returned as string for precision.     |
| `total_score`    | Number or null  | `8.3`                                     | Composite risk score 0-10. Null means unrated.                   |
| `risk_tier`      | String          | `"Prime"`                                 | Derived from total_score: Prime (>=8), Core (5-7.9), Edge (<5).  |
| `risk_vectors`   | Object          | `{"asset":{"score":8.5},...}`             | Breakdown: asset (40%), platform (40%), control (20%).           |
| `is_audited`     | Boolean         | `true`                                    | Whether the protocol version has been audited.                   |
| `last_synced_at` | ISO 8601        | `"2026-02-26T12:00:00Z"`                  | When the vault data was last refreshed from on-chain.            |
| `rewards`        | Array           | `[{"token_symbol":"MORPHO","apr":0.018}]` | Individual reward token APRs. Sum + base_apr = apr_net.          |

## Error Handling

The CLI automatically retries failed requests (network errors, 429, 5xx) up to 3 times with exponential backoff. Exit codes indicate error type:

| Exit Code | Meaning          | Cause                            |
| --------- | ---------------- | -------------------------------- |
| 0         | Success          | Normal exit                      |
| 1         | General error    | Unknown / unexpected             |
| 2         | Network error    | Connection refused, DNS, timeout |
| 3         | Client error     | 400, 404, 422 (bad request)      |
| 4         | Server error     | 5xx / 429 after retries          |
| 5         | Validation error | Invalid CLI arguments            |

| Error                   | Cause                                     | Fix                                                                                                                      |
| ----------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `Connection refused`    | API server unreachable                    | Check network connectivity. Verify `--api-url` if using a custom endpoint. The default is `https://api.philidor.io`.     |
| `404 Not Found`         | Invalid vault ID, protocol ID, or address | Verify the vault ID exists with `philidor vaults`. Check that the network slug is correct (e.g., `ethereum`, not `eth`). |
| `429 Too Many Requests` | Rate limit exceeded                       | The CLI retries automatically. If still failing, space out bulk queries.                                                 |
| `Request timeout`       | API response took too long                | The CLI retries automatically. For large result sets, use `--limit` to reduce page size.                                 |
| `Invalid chain`         | Unsupported chain slug                    | Run `philidor chains` to see valid chain slugs.                                                                          |

## Best Practices

1. **Always use `--json` in agent workflows.** JSON output is structured, stable, and machine-parseable. Table output is for human consumption only and may change format between versions.

2. **Check risk before recommending any vault.** Never recommend a vault based on APR alone. Always run `philidor risk breakdown <id>` and check for Edge-tier scores or recent incidents via `philidor risk incidents`.

3. **Note data freshness timestamps.** The `last_synced_at` field indicates when vault data was last refreshed from on-chain sources. Data older than a few hours may not reflect current APR or TVL. Flag stale data to users.

4. **Cross-reference incidents and oracle health.** Before recommending a vault, check `philidor risk incidents` for active security events and `philidor oracles freshness` for stale price feeds. These can materially affect vault safety.

5. **Use portfolio context for personalised advice.** When a user has a wallet address, start with `philidor portfolio <address>` to understand their existing positions before suggesting new vaults. This enables risk-aware recommendations that consider concentration and diversification.

## Resources

- **Website**: [https://philidor.io](https://philidor.io)
- **API Documentation**: [https://api.philidor.io/v1/docs](https://api.philidor.io/v1/docs)
- **Risk Methodology**: `philidor risk explain` or [https://philidor.io/risk](https://philidor.io/risk)
