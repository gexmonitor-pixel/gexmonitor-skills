---
name: gexmonitor
description: Use GEX Monitor for crypto derivatives and options market-structure research, including BTC/ETH/SOL market snapshots, Gamma/GEX, volatility, options flow, whale activity, expiry attribution, daily observations, market-structure comparisons, and option-strategy analysis.
---

# GEX Monitor

GEX Monitor provides authenticated live crypto options and derivatives market-structure research through its Remote MCP server.

Canonical endpoint:
`https://gexmonitor.com/api/mcp`

The Skill does not contain market data itself. Always use the live MCP tools when current market information is required.

---

## When to Use

Invoke GEX Monitor when the user asks about:
- BTC / ETH / SOL options and derivatives structure
- Gamma Exposure (GEX) and Net GEX regimes
- Gamma Flip levels and distance metrics
- Call Wall and Put Wall locations and concentrations
- Dealer hedging dynamics and positioning shifts
- Implied Volatility (IV), Realized Volatility (RV), and Volatility Risk Premium (VRP)
- Volatility term structure and 25-delta / 10-delta Skew (RR25)
- Options flow volume, turnover, and buy/sell aggression breakdown
- Whale activity, large block trades, and smart-money Open Interest (OI) actions
- Expiry attribution, structural maturity weight, and roll/pin risks
- Daily editorial market observations and publication snapshots
- Cross-asset market structure comparisons between BTC and ETH
- Option strategy economics, Greek exposure, and payoff scenarios

**Do NOT** invoke GEX Monitor for general cryptocurrency price queries, basic spot conversions, meme coins, or unrelated technical topics.

---

## MCP Connection & Authentication

1. **Active Connection**: If the host agent already has access to the GEX Monitor MCP tools, execute them directly.
2. **Missing Connection**: If not connected, inform the user that GEX Monitor Remote MCP is accessible at:
   `https://gexmonitor.com/api/mcp`
   and guide them to use their agent's native Remote MCP integration flow.
3. **Authentication**: Identity is handled via the remote server's OAuth 2.0 / dynamic client registration flow.
   - **Never** prompt the user to manually supply client IDs, client secrets, database passwords, Supabase keys, or bearer tokens.
4. **Unsupported Agents**: If the host agent does not support Remote MCP servers, state this limitation clearly. Never fabricate tool responses.

---

## Research Tool Surface

GEX Monitor exposes exactly **9 canonical Research Tools**. Do not invent additional tools or parameters:

| Tool | Purpose | Primary Inputs |
| :--- | :--- | :--- |
| `get_market_snapshot` | Current canonical spot & core Gamma structure | `asset`: `"BTC"` \| `"ETH"` \| `"SOL"` |
| `get_gamma_snapshot` | Strike-level GEX distribution & major levels | `asset`: `"BTC"` \| `"ETH"` \| `"SOL"` |
| `get_volatility_snapshot` | ATM IV, RV, VRP, Skew (25d/10d), & IV percentile | `asset`: `"BTC"` \| `"ETH"` \| `"SOL"` |
| `get_options_flow` | Aggregate taker volume, turnover & aggression | `asset`: `"BTC"` \| `"ETH"`, `time_range`: `"1h"` \| `"4h"` \| `"24h"` |
| `get_whale_activity` | Institutional block trades & OI action breakdown | `asset`: `"BTC"` \| `"ETH"`, `timeframe`: `"1h"` \| `"4h"` \| `"24h"`, `limit`?: 1–50 |
| `get_expiry_attribution` | Per-expiry Gamma attribution & maturity weight | `asset`: `"BTC"` \| `"ETH"` \| `"SOL"` |
| `get_daily_observation` | Editorial morning brief & publication snapshot | `asset`: `"BTC"` \| `"ETH"`, `locale`: `"en"` \| `"zh-CN"` \| `"zh-TW"` |
| `compare_market_structure`| Neutral side-by-side BTC vs. ETH comparison | `assets`: `["BTC", "ETH"]` |
| `analyze_option_strategy` | Deterministic strategy quote payoff & Greeks | `asset`, `expiry`, `strategy_type`, `legs`, `user_target_price`? |

Detailed schemas, input parameters, and output contracts are documented in [references/tools.md](references/tools.md). The runtime MCP schema remains authoritative.

---

## Efficient Tool Routing

Always select the smallest sufficient tool set for the user's inquiry:

- *"What is BTC doing right now?"* → `get_market_snapshot`
- *"Where are the critical Gamma levels?"* → `get_gamma_snapshot`
- *"Is volatility expensive or cheap?"* → `get_volatility_snapshot`
- *"Are options traders buying calls or puts?"* → `get_options_flow`
- *"What are whales doing in options?"* → `get_whale_activity`
- *"Which expiry controls the market?"* → `get_expiry_attribution`
- *"Show me today's full market brief"* → `get_daily_observation`
- *"How is BTC structure different from ETH?"* → `compare_market_structure`
- *"Analyze a BTC 95k/105k bull call spread"* → `analyze_option_strategy`

For multi-step inquiries, follow the structured sequences documented in [references/workflows.md](references/workflows.md).

---

## Core Interpretation Principles

When synthesizing research, adhere strictly to GEX Monitor methodology:

1. **Price Action Has Priority**: Market price movement supersedes model assumptions. If spot breaches a Wall, report the breakout rather than asserting the model is flawed.
2. **Walls Are Sensitivity Areas**: Call Walls and Put Walls mark dealer hedging inflection zones, never guaranteed support or resistance.
3. **Volume $\neq$ Trade Direction**: High trading volume does not automatically imply buying pressure. Inspect trade side, aggression, and OI changes.
4. **Activity $\neq$ Conviction**: High turnover frequently reflects delta-neutral rebalancing or spread execution rather than directional speculation.
5. **Separate Analytical Layers**:
   - **Observed Data**: Factual figures returned directly by tools.
   - **Structural Interpretation**: Mechanism-based analysis of dealer hedging incentives.
   - **Conditional Inference**: Scenarios contingent on specific triggers.
6. **No Deterministic Predictions**: Avoid statements like "BTC will definitely rise" or "the Put Wall must hold."
7. **Focus on Mechanisms**: Emphasize where positioning changes hedging pressure, where volatility is being repriced, and what invalidates the outlook.
8. **Terminology Precision**: Use the term **"path"** (or in Chinese: **"路径"**). Never use the Chinese term **"剧本"**.
9. **Conditional Phrasing**: Prefer formulations like *"If spot remains above [level], the structure is consistent with..."* and *"This interpretation weakens if..."*.

See [references/data-contract.md](references/data-contract.md) for full interpretation principles and freshness contracts.

---

## Data Freshness & Degradation

Inspect response metadata on every call:
- **`meta.availability`**:
  - `ready`: Full data integrity.
  - `partial`: Usable, but specific peripheral fields are degraded; disclose missing items to the user.
  - `missing`: Source unavailable; acknowledge missing data truthfully without inventing numbers.
- **`meta.stale`**: When `true`, explicitly qualify findings with the `stale_reason` and quote timestamp.
- **Recency vs. Freshness**: Do not confuse `pipeline_status` with `activity_status`. If `activity_status="NO_WHALE_ACTIVITY"`, this reflects an absence of large trades in that window, not an infrastructure outage.

---

## Output Structuring

For broad, multi-tool market-structure requests, format responses into this standard research structure:

1. **One-Sentence Conclusion**: High-level structural stance.
2. **Current Structural Bias**: Operating regime (e.g., long vs. short dealer gamma, volatility state).
3. **What Changed**: Shifts in levels, flow, or skew since recent sessions.
4. **Key Levels / Zones**: Current spot, Gamma Flip, Call Wall, Put Wall, and distances.
5. **Evidence**: Concrete supporting metrics from tools (flow turnover, VRP, whale trades).
6. **What to Watch Next**: Catalysts, upcoming expiries, or volume nodes.
7. **Invalidation Criteria**: Specific price levels or flow divergences that invalidate the structural view.

*Do not force this structured format onto simple, single-metric factual questions.*

### Language Policy
- **English**: Respond in professional financial English.
- **Chinese**: When queried in Chinese, respond primarily in Chinese while preserving standard industry terms in English (`Delta`, `Gamma`, `Vega`, `Theta`, `IV`, `RV`, `VRP`, `Skew`, `RR25`, `GEX`, `Net GEX`, `Call Wall`, `Put Wall`, `Max Pain`, `DTE`).
- Standard definitions are available in [references/terminology.md](references/terminology.md).

---

## Entitlement & Auth Failures

If an MCP tool returns `UNAUTHORIZED` or `FORBIDDEN`:
- Inform the user succintly that GEX Monitor PRO entitlement or authentication is required.
- Do not make repeated automated retry attempts.
- Do not attempt to bypass access restrictions or prompt the user for secret tokens.

---

## Security Guardrails

Agents utilizing this skill must strictly observe these operational boundaries:
- **Prohibited**: Logging, requesting, or transmitting OAuth tokens, authorization codes, Supabase keys, or API secrets.
- **Prohibited**: Bypassing entitlement checks, attempting direct database (SQL) connections, accessing Redis caches, or scraping private web paths.
- **Prohibited**: Inventing unsupported MCP methods or fabricating analytical numbers.
- **Prohibited**: Attempting order placement or trading execution. GEX Monitor is exclusively a research and analytical intelligence tool.
