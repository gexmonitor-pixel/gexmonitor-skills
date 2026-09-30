# GEX Monitor Multi-Tool Research Workflows

To conduct rigorous market-structure research, agents should combine individual tools into structured analytical sequences. Avoid executing arbitrary tool combinations; use the smallest sufficient tool set for the user's objective.

---

## 1. Full Market Structure Analysis

Comprehensive analysis of an asset's complete derivatives posture.

### Sequence
```
1. get_market_snapshot
   └── Establish baseline: Spot, Net GEX regime, distance to Flip and Walls
2. get_gamma_snapshot
   └── Map strike concentrations: Major gamma clusters, wall thicknesses
3. get_volatility_snapshot
   └── Assess pricing environment: IV percentile, RV, VRP, 25d Skew
4. get_options_flow (24h)
   └── Inspect market momentum: Call vs. Put turnover, aggressive taker buying/selling
5. get_whale_activity (24h)
   └── Inspect institutional commitment: Block prints, OI opening vs. closing
6. get_expiry_attribution (conditional: if near major weekly/monthly expiry)
   └── Gauge maturity distribution: Pin risk, gamma cliff, roll weight
```

### Synthesis Guide
- Check if spot is operating in positive (volatility-dampening) or negative (volatility-amplifying) gamma regime.
- Compare options flow direction with current spot trend: Is spot advancing on aggressive call buying (organic tailwind) or diverging?
- Determine if institutional whale prints are opening new directional risk (`confirmed_open`) or hedging existing books (`confirmed_close`).
- Identify the primary invalidation level (typically the Gamma Flip or closest major Wall).

---

## 2. Breakout / Breakdown Research

Evaluate whether an impending or active price move through a key level has structural backing.

### Sequence
```
1. get_market_snapshot
   └── Identify distance from spot to Call Wall or Put Wall
2. get_gamma_snapshot
   └── Quantify the gamma concentration at the breached level
3. get_options_flow (1h or 4h)
   └── Determine whether flow is actively chasing the move or fading it
4. get_volatility_snapshot
   └── Inspect skew and IV: Is volatility expanding (momentum breakout) or compressing?
```

### Analytical Objective
Determine whether price action is supported by structural positioning, volatility repricing, and aggressive flow:
- **True Structural Breakout Profile**: Spot moves through a Wall while short-term flow shows aggressive taker call buying, IV expands, and positive skew steepens.
- **Exhaustion / Reversal Setup**: Spot touches a Wall while flow weakens, IV fails to expand, and dealer positive gamma dampens upward acceleration.
- **Rule**: Never assert certainty. State conditional paths: *"If flow continues to absorb the Call Wall and IV expands, the structure favors continuation toward [next major level]. If spot drops back below [wall], the move degrades into a failed test."*

---

## 3. Event & Expiry Research

Analyze market vulnerability and dealer positioning heading into major macro releases (CPI, FOMC) or monthly/quarterly contract settlement.

### Sequence
```
1. get_market_snapshot
   └── Establish current spot and baseline gamma boundaries
2. get_volatility_snapshot
   └── Evaluate event risk premium: Is VRP elevated? Is term structure inverted?
3. get_expiry_attribution
   └── Identify the dominant expiry holding the bulk of total market gamma
4. get_gamma_snapshot
   └── Locate strike clusters and the volatility trigger level
5. get_options_flow (4h or 24h)
   └── Track pre-event hedging activity or defensive put accumulation
```

### Analytical Objective
- Identify upcoming **Gamma Cliffs**: When a large expiry holds >40% of total market gamma, its expiration removes substantial dealer hedging pressure, frequently unlocking higher spot volatility in subsequent sessions.
- Pin Risk: If spot trades near major strikes of the expiring cycle with heavy positive gamma, dealers are incentivized to keep spot pinned to minimize hedge slippage.

---

## 4. Option Strategy Research

Evaluate a proposed trading structure against live market quotes and prevailing market regime.

### Sequence
```
1. get_market_snapshot
   └── Locate where the strategy's strikes sit relative to the Gamma Flip and Walls
2. get_volatility_snapshot
   └── Check IV percentile and VRP to assess whether options pricing favors debit or credit structures
3. get_gamma_snapshot (optional/contextual)
   └── Confirm dealer hedging dynamics around the strategy strikes
4. analyze_option_strategy
   └── Compute exact bid/ask entry cost, maximum profit/loss, breakevens, Greeks, and scenario payoffs
```

### Analytical Objective
- **Debit Strategies** (e.g., Bull Call Spreads, Long Calls): Favor low IV percentile, compressed VRP, and spots positioned to benefit from negative gamma momentum.
- **Credit Strategies** (e.g., Bull Put Spreads, Short Strangles): Favor high IV percentile, rich positive VRP, and spot contained within thick positive gamma walls.
- Synthesize the mathematical results from `analyze_option_strategy` with the prevailing volatility and gamma environment without providing personalized investment advice.

---

## 5. "Why Is Today Different?" & Cross-Asset Analysis

Explain recent structural shifts or divergences between BTC and ETH.

### Sequence
```
1. get_daily_observation
   └── Read published morning baseline and recorded publication-time snapshot
2. compare_market_structure
   └── Directly align BTC and ETH across spot, gamma, IV, VRP, and flow metrics
3. get_market_snapshot (for asset of interest)
   └── Measure exact intraday drift relative to the morning observation snapshot
4. get_options_flow / get_whale_activity (selective drilldown)
   └── Isolate the specific driver (e.g., massive institutional block trade or IV crush)
```

### Analytical Objective
- Pinpoint exactly what changed between the morning baseline and current live conditions:
  - Did spot cross the Gamma Flip?
  - Did the Call Wall shift higher or lower?
  - Did VRP collapse or spike?
  - Did whale participants enter large closing transactions?
