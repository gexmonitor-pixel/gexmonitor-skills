# GEX Monitor Research Tools Reference

GEX Monitor publishes exactly **9 canonical Research Tools** via its Remote MCP server. These tools provide factual, authenticated crypto options and derivatives market-structure data.

> [!NOTE]
> MCP tool schema is authoritative at runtime. Tool execution is strictly read-only and analytical. No trade execution capabilities exist.

---

## 1. `get_market_snapshot`

### Purpose
Reads the current canonical spot price and core Gamma market structure for an asset. Serves as the primary orientation tool.

### Inputs
```json
{
  "asset": "BTC" // "BTC" | "ETH" | "SOL" (required)
}
```

### Outputs
- `spot_price`: Current underlying reference spot price.
- `gamma_flip`: Price level separating positive and negative gamma regimes.
- `call_wall`: Strike with highest call gamma concentration.
- `put_wall`: Strike with highest put gamma concentration.
- `net_gex`: Total aggregate dollar gamma exposure.
- `gamma_regime`: Descriptive regime tag (e.g., `"positive_gamma"`, `"negative_gamma"`).
- `distance_to_flip_pct`: Spot-relative percentage distance: `((spot - gamma_flip) / spot) * 100`.
- `distance_to_call_wall_pct`: Percentage distance from spot to Call Wall.
- `distance_to_put_wall_pct`: Percentage distance from spot to Put Wall.

### Typical User Questions
- "What is BTC doing right now?"
- "Where is the Gamma Flip on ETH?"
- "How close is BTC to its Call Wall?"

### Relation to Other Tools
The foundational starting point for almost every workflow. Precedes `get_gamma_snapshot` for strike-level details or `get_volatility_snapshot` for pricing context.

### Interpretation Caveats
Walls are sensitivity zones and hedging inflection points, not guaranteed support or resistance. Distances are signed percentages from current spot.

---

## 2. `get_gamma_snapshot`

### Purpose
Delivers a deep structural breakdown of dealer Gamma exposure across strikes, including the top major gamma levels and volatility indicators.

### Inputs
```json
{
  "asset": "BTC" // "BTC" | "ETH" | "SOL" (required)
}
```

### Outputs
- `spot`: Reference spot price.
- `gamma_flip`: Gamma flip strike.
- `gamma_zone`: Descriptive zone categorization.
- `net_gex`, `call_gex`, `put_gex`: Aggregate dollar Greek exposures.
- `call_wall`, `put_wall`: Wall structures containing strike, value, and distance percentage.
- `vol_trigger`: Volatility acceleration trigger price.
- `dvol`: Deribit Volatility Index level.
- `iv_percentile`: Trailing IV percentile ranking (0–100 scale).
- `major_gamma_levels`: Array of up to 8 top strikes with individual `strike`, `net_gex`, `call_gex`, `put_gex`, and `distance_pct`.

### Typical User Questions
- "Where are the major gamma concentrations for BTC?"
- "What happens to dealer hedging if BTC drops below $92,000?"
- "Show me the strike-by-strike GEX distribution."

### Relation to Other Tools
Expands upon `get_market_snapshot` with granular strike clusters. Supplies structural context for `get_options_flow` and `analyze_option_strategy`.

### Interpretation Caveats
In a positive gamma regime, dealer rebalancing dampens market volatility (buying dips, selling rallies). In a negative gamma regime, dealer hedging amplifies spot momentum (selling breaks, buying rips).

---

## 3. `get_volatility_snapshot`

### Purpose
Supplies current implied volatility, realized volatility, volatility risk premium (VRP), and skew metrics.

### Inputs
```json
{
  "asset": "BTC" // "BTC" | "ETH" | "SOL" (required)
}
```

### Outputs
- `atm_iv`: At-the-money implied volatility level.
- `iv_percentile`: Percentile rank of current IV against historical window (0–100).
- `iv_rank`: Relative rank within the 52-week IV range.
- `rv`: Trailing 30-day realized volatility.
- `vrp`: Volatility Risk Premium ($\text{VRP} = \text{IV} - \text{RV}_{30\text{d}}$).
- `dvol`: 30-day constant-maturity options-implied volatility index.
- `skew_25d`: 25-delta Risk Reversal skew ($\text{IV}_{\text{Call}} - \text{IV}_{\text{Put}}$).
- `skew_10d`: 10-delta Risk Reversal skew for tail-risk analysis.

### Typical User Questions
- "Is BTC options IV expensive right now?"
- "What is the current VRP on ETH?"
- "Is the market pricing upside skew or downside tail protection?"

### Relation to Other Tools
Critical companion to `get_gamma_snapshot`. Must be consulted prior to `analyze_option_strategy` to determine whether volatility environment favors net long or net short option structures.

### Interpretation Caveats
A high IV percentile does not automatically warrant selling premium if imminent macro catalysts are pending. Elevated negative skew signifies pronounced demand for out-of-the-money put protection.

---

## 4. `get_options_flow`

### Purpose
Aggregates recent options taker execution activity and volume turnover across 1h, 4h, or 24h rolling windows for BTC or ETH.

### Inputs
```json
{
  "asset": "BTC", // "BTC" | "ETH" (required)
  "time_range": "24h" // "1h" | "4h" | "24h" (required)
}
```

### Outputs
- `asset`: Target asset.
- `time_range`: Filter window (`"1h"`, `"4h"`, `"24h"`).
- `summary`: Aggregate metrics including total trade count, turnover premium (USD), notional volume (USD), call/put volume ratios, and aggressor side breakdown (buy vs. sell).
- `top_concentrations`: Strikes and expirations attracting the highest turnover and trade counts.

### Typical User Questions
- "Are options traders buying calls or puts today?"
- "What does the 4-hour options flow look like for ETH?"
- "Where is the largest options volume landing?"

### Relation to Other Tools
Pairs directly with `get_whale_activity` to distinguish broad market retail/algo flow from institutional block prints.

### Interpretation Caveats
High flow volume alone does not indicate directional conviction. It frequently reflects delta-neutral spreads, calendar rolls, or closing existing positions. Always inspect trade side and strike concentration.

---

## 5. `get_whale_activity`

### Purpose
Retrieves large institutional prints, block trades, and smart-money options positioning for BTC or ETH over 1h, 4h, or 24h windows.

### Inputs
```json
{
  "asset": "BTC", // "BTC" | "ETH" (required)
  "timeframe": "24h", // "1h" | "4h" | "24h" (required)
  "limit": 5 // Optional integer between 1 and 50 (default: 5)
}
```

### Outputs
- `timeframe`: Query window (`"1h"`, `"4h"`, `"24h"`).
- `pipeline_status`: Data pipeline health (`"healthy"`, `"degraded"`, `"fail_close"`).
- `activity_status`: State indicator (`"WHALE_ACTIVITY_AVAILABLE"`, `"NO_WHALE_ACTIVITY"`, `"WHALE_DATA_STALE"`).
- `summary`: Premium turnover, notional volume, call/put count, largest single print, and Open Interest action breakdown (`confirmed_open`, `confirmed_close`, `inferred_open`, `inferred_close`).
- `trades`: Array of individual transactions with timestamp, instrument, strike, expiry, trade side (buy/sell), is_block, premium, notional, and Greeks.

### Typical User Questions
- "What are whales doing in BTC options?"
- "Were there any massive block trades in the last 4 hours?"
- "Are institutional players opening new calls or closing puts?"

### Relation to Other Tools
Validates whether gamma accumulation at key walls is driven by institutional opening orders or retail distribution.

### Interpretation Caveats
Large trades are frequently delta-hedged against perpetuals or spot. An `activity_status: "NO_WHALE_ACTIVITY"` indicates no qualifying prints occurred during the window—not a pipeline failure.

---

## 6. `get_expiry_attribution`

### Purpose
Attributes total market Gamma exposure across calendar expiration dates, revealing maturity concentration and upcoming roll/pin risks.

### Inputs
```json
{
  "asset": "BTC" // "BTC" | "ETH" | "SOL" (required)
}
```

### Outputs
- Expiry list sorted by structural weight:
  - `expiry`: Calendar date (`"YYYY-MM-DD"`).
  - `expiry_label`: Formatted label (e.g., `"27-MAR-26"`).
  - `dte`: Days to expiration.
  - `net_gex`, `call_gex`, `put_gex`, `absolute_gex`: Greek attribution.
  - `structural_weight_pct`: Percentage share of total market absolute gamma.
  - `rank`: Relative maturity weight rank.
  - `dominant_strikes`: Top strike levels anchoring gamma for this specific expiry.

### Typical User Questions
- "Which options expiry holds the most gamma?"
- "How much dealer gamma expires this Friday?"
- "What is the structural weight of the quarterly expiry?"

### Relation to Other Tools
Explains shifts in `get_market_snapshot` and `get_gamma_snapshot` as major expiries approach or roll off.

### Interpretation Caveats
Gamma concentration accelerates dramatically near 0 DTE due to rapid Gamma expansion, but aggregate dollar risk may quickly transfer to subsequent expiries as institutional desks roll positions.

---

## 7. `get_daily_observation`

### Purpose
Fetches the editorial market-structure commentary and publication-time snapshot published daily by GEX Monitor research analysts.

### Inputs
```json
{
  "asset": "BTC", // "BTC" | "ETH" (required)
  "locale": "en" // "en" | "zh-CN" | "zh-TW" (required)
}
```

### Outputs
- `report_date`: Observation date.
- `edition`: Edition tag (`"main"`).
- `published_at`: Publication UTC timestamp.
- `content`: Full editorial synthesis written by market analysts.
- `snapshot`: Publication-time benchmark values: spot, gamma flip, net GEX, DVOL, put/call ratio, Call Wall, Put Wall.

### Typical User Questions
- "What was today's daily observation for BTC?"
- "Summarize the morning market brief."
- "How did analysts view the market structure at daily open?"

### Relation to Other Tools
Ideal orientation tool when a user seeks an overarching narrative before drilling into live real-time metrics.

### Interpretation Caveats
Represents conditions at the time of morning publication. Market structure may have evolved since release; verify against `get_market_snapshot` for live conditions.

---

## 8. `compare_market_structure`

### Purpose
Performs a factual, multi-dimensional comparison of market structure between BTC and ETH across spot, gamma, volatility, flow, and whale metrics.

### Inputs
```json
{
  "assets": ["BTC", "ETH"] // Strict two-asset array containing exactly "BTC" and "ETH"
}
```

### Outputs
- Side-by-side comparison object contrasting:
  - Spot prices and distances to Gamma Flip and Walls.
  - Net GEX regimes and absolute exposures.
  - Implied volatility, Realized volatility, and VRP spreads.
  - 24h Options Flow volume and call/put ratios.
  - Whale trade counts and institutional turnover.

### Typical User Questions
- "How does BTC market structure compare to ETH?"
- "Is ETH volatility trading at a steeper premium than BTC?"
- "Which asset shows stronger call flow today?"

### Relation to Other Tools
Orchestrates and compares outputs from `get_market_snapshot`, `get_volatility_snapshot`, `get_options_flow`, and `get_whale_activity`.

### Interpretation Caveats
Maintains strict neutrality: does not rank assets, generate conviction scores, or recommend pair trades.

---

## 9. `analyze_option_strategy`

### Purpose
Evaluates deterministic economics, Greek exposures, and scenario payoffs for standard single-leg and two-leg option structures using live canonical quotes.

### Inputs
```json
{
  "asset": "BTC", // "BTC" | "ETH" | "SOL" (required)
  "expiry": "30OCT26", // Expiry in DMMMYY or DDMMMYY format (required)
  "strategy_type": "bull_call_spread", // Supported strategy enum (required)
  "legs": [
    { "strike": 95000, "option_type": "call", "side": "buy" },
    { "strike": 105000, "option_type": "call", "side": "sell" }
  ],
  "user_target_price": 102000 // Optional hypothetical spot target
}
```

#### Supported Strategy Types
- Single Leg: `long_call`, `long_put`, `short_call`, `short_put`
- Vertical Spreads: `bull_call_spread`, `bear_put_spread`, `bull_put_spread`, `bear_call_spread`
- Combinations: `long_straddle`, `long_strangle`, `short_straddle`, `short_strangle`

### Outputs
- Entry pricing: Net debit or credit, quote timestamps, bid/ask spreads.
- Boundary metrics: Maximum theoretical profit, maximum theoretical loss, risk/reward ratio.
- Breakevens: Strike-derived zero-profit underlying prices at expiration.
- Aggregate Greeks: Position Delta, Gamma, Vega, and Theta.
- Scenarios: Payoff matrix across varying underlying spot movements and DTE steps.

### Typical User Questions
- "Analyze a BTC 95k/105k bull call spread for next month."
- "What is the breakeven and max loss on this ETH straddle?"
- "How does time decay affect this spread if spot stays flat?"

### Relation to Other Tools
Should follow `get_volatility_snapshot` and `get_gamma_snapshot` to ensure the strategy matches the prevailing volatility regime and dealer positioning.

### Interpretation Caveats
Purely deterministic quote-based calculation. Returns factual scenario models, not trade recommendations or probabilistic assurances.
