# GEX Monitor Data Freshness & Interpretation Contract

This document details how AI agents must parse GEX Monitor response envelopes, evaluate data freshness, handle degraded states, and adhere to core market-structure interpretation principles.

---

## 1. Response Envelope Structure

Every GEX Monitor research tool response is structured in a standardized envelope:

```json
{
  "tool": "get_market_snapshot",
  "data": { ... },
  "meta": {
    "asset": "BTC",
    "as_of": "2026-09-30T10:45:00.000Z",
    "generated_at": "2026-09-30T10:47:12.345Z",
    "stale": false,
    "stale_reason": null,
    "sources": ["gex_profile_snapshot"],
    "availability": "ready"
  },
  "errors": []
}
```

### Core Envelope Fields

- **`meta.availability`**: The canonical indicator of data completeness. Permitted values:
  - `ready`: All required data sources loaded successfully with full integrity.
  - `partial`: Primary computation succeeded, but one or more peripheral metrics (e.g., secondary IV metric or specific strike level) are missing or degraded. The agent may use the available fields but must disclose degraded metrics.
  - `missing`: The requested dataset could not be produced (e.g., upstream calculation failure or unpopulated cache). `data` will be `null`.
- **`meta.stale`** (`boolean`): True if computation age exceeds canonical cache freshness thresholds (e.g., 15 minutes for GEX profiles, 3 minutes for options flow summaries).
- **`meta.as_of`** / **`meta.data_timestamp`**: Timestamp of the latest underlying market quote or trade included in the calculation.
- **`meta.generated_at`** / **`meta.computed_at`**: Timestamp when the server calculated or packaged the snapshot.
- **`meta.stale_reason`**: Explicit description of why the payload is marked stale when `stale: true`.
- **`errors`**: Array of structured issues (`ResearchToolIssue`):
  - `code`: `UNAUTHORIZED`, `FORBIDDEN`, `INVALID_INPUT`, `DATA_UNAVAILABLE`, `DATA_STALE`, `UPSTREAM_ERROR`, `INTERNAL_ERROR`.
  - `severity`: `warning` vs. `error`.
  - `field`: The specific field that failed or is degraded, if applicable.

---

## 2. Freshness vs. Market-Event Recency

Agents must maintain a strict conceptual distinction between **computation freshness** and **market-event recency**:

| Dimension | Metric / Flag | Meaning | Interpretation |
| :--- | :--- | :--- | :--- |
| **Pipeline Freshness** | `stale: false`<br>`pipeline_status: "healthy"`<br>`pipeline_heartbeat_at` | The backend calculation engine and data ingestion pipelines are operating normally. | System is healthy and operating within SLA. |
| **Event Recency** | `event_stale: true`<br>`activity_status: "NO_WHALE_ACTIVITY"`<br>`latest_trade_at` | No trade or qualifying market action occurred within the queried window. | **Natural market condition**, not a technical failure. During quiet market hours, whale prints or large block trades may legitimately not occur for hours. |

> [!IMPORTANT]
> If `event_stale=true` or `activity_status="NO_WHALE_ACTIVITY"`, do **not** report an infrastructure outage. Explain truthfully that market participants have executed no qualifying transactions during the observed period.

---

## 3. Degradation Rules

1. **Explicit Degradation**: When `availability: "partial"`, state clearly which fields are missing or degraded. Never silently replace missing values with assumed numbers or zeroes.
2. **Explicit Failure**: When `availability: "missing"` or `data: null`, acknowledge that current data is unavailable. Never manufacture artificial Gamma levels, spot prices, or flow figures.
3. **Stale Handling**: When `stale: true`, present the data with an explicit qualification: "Note: Data as of [as_of] is stale ([stale_reason]). Market conditions may have shifted."

---

## 4. GEX Monitor Interpretation Principles

When synthesizing research outputs from tool data, agents must strictly follow these ten core analytical principles:

1. **Price Behavior Has Priority Over Model Interpretation**  
   Market price action is primary. Models, Greek exposures, and Walls describe structural conditions and sensitivities, not immutable barriers. If price cleanly trades through a Wall, observe the breakout rather than asserting the model was "wrong" or the market "must reverse."

2. **Walls Are Sensitivity Areas, Not Guaranteed Support/Resistance**  
   Call Walls and Put Walls mark strikes with high gamma concentrations where dealer hedging delta swings sharply. They represent pivot zones and sensitivity areas, never guaranteed floors or ceilings.

3. **Volume Does Not Equal Trade Direction**  
   Large trading volume does not automatically indicate aggressive buying. Always cross-reference trade aggressor side (`trade_side`), quote spread, and open interest changes (`oi_action`) before drawing directional inferences.

4. **High Activity Does Not Imply Directional Conviction**  
   Surges in options turnover frequently reflect delta-neutral hedging, calendar roll execution, collateral management, or spread legs rather than directional speculation.

5. **Strictly Distinguish the Three Analytical Layers**:
   - **Observed Data**: Factual metrics returned directly by tools (e.g., Spot at \$95,400, Net GEX of -\$42M, 24h Call Volume \$120M).
   - **Structural Interpretation**: Mechanism-based deduction (e.g., "Dealers are in a negative gamma regime below \$96,000, which historically accelerates spot volatility").
   - **Conditional Inference**: Scenarios contingent on triggers (e.g., "If spot remains below the Gamma Flip, dealer short-gamma hedging pressure is expected to amplify downside momentum").

6. **Never Present Market Structure as Certainty**  
   Avoid deterministic phrasing such as "BTC will definitely rally", "the Put Wall will hold", or "dealers must dump spot." Market makers hedge dynamically across options, perps, spot, and synthetic instruments.

7. **Focus on Four Structural Drivers**:
   - Where positioning can alter dealer hedging behavior (Gamma Flip and major Wall boundaries).
   - Where volatility is being repriced (IV shifts, Skew steepening, VRP expansions/compressions).
   - Where flow confirms or contradicts observed price action.
   - What critical levels would invalidate the current structural interpretation.

8. **Terminology Precision**:
   - Use the word **"path"** (or in Chinese: **"路径"**).
   - **Never** use the Chinese term **"剧本"**. Market dynamics unfold along probabilistic paths, not scripted theatrics.

9. **Avoid Deterministic Predictions**  
   Never issue target price guarantees, buy/sell recommendations, or promises of wall integrity.

10. **Use Conditional, Evidence-Backed Phrasing**:
    - *"If spot sustains above [level], the structure favors..."*
    - *"The current distribution is consistent with..."*
    - *"This interpretation weakens if flow diverges or price breaks below [invalidation level]..."*
