# GEX Monitor Terminology Reference

A concise, professional glossary of derivatives, market-structure, and options Greeks terminology used across GEX Monitor research tools and analytics.

---

### Greeks & Volatility

- **Delta**: The rate of change of option price with respect to changes in the underlying asset price ($\partial V / \partial S$). Also represents the directional hedge ratio required by market makers per contract.
- **Gamma**: The second derivative of option price with respect to underlying spot price ($\partial^2 V / \partial S^2$), representing the rate of change of Delta. Quantifies the acceleration of dealer hedging requirements as spot moves.
- **Vega**: Sensitivity of option price to a 1-percentage-point change in implied volatility ($\partial V / \partial \sigma$).
- **Theta**: The rate of option value decay over time ($\partial V / \partial t$), reflecting calendar decay holding spot and volatility constant.
- **IV (Implied Volatility)**: The annualized standard deviation of expected underlying price movements implied by market option quotes via option pricing models (e.g., Black-76).
- **RV (Realized Volatility)**: The annualized historical standard deviation of actual underlying logarithmic asset returns over a trailing sample window (e.g., trailing 7-day or 30-day).
- **VRP (Volatility Risk Premium)**: The spread between implied volatility and realized volatility, formulated in GEX Monitor as $\text{VRP} = \text{IV} - \text{RV}_{30\text{d}}$. A positive VRP indicates implied volatility trades at a premium to realized volatility, reflecting the variance risk compensation demanded by option sellers.
- **Skew**: Asymmetry in implied volatility across strike prices within the same expiration cycle, reflecting market pricing for asymmetric tail risk.
- **RR25 (25-Delta Risk Reversal)**: Implied volatility spread between the 25-delta call and the 25-delta put ($\text{IV}_{25\text{d C}} - \text{IV}_{25\text{d P}}$). Positive RR25 reflects call premium (upside skew), while negative RR25 reflects put premium (downside protection demand).
- **Term Structure**: The curve mapping implied volatility across consecutive expiration dates (DTE). Upward-sloping curves represent contango (longer maturities price higher uncertainty); downward-sloping curves represent backwardation (front-month event risk or near-term panic).

---

### Gamma & Positioning Metrics

- **GEX (Gamma Exposure)**: Modeled dollar-value rate of change in dealer aggregate delta per 1% move in underlying spot price, assuming market makers provide two-sided liquidity and absorb customer order flow.
- **Net GEX**: The net sum of call gamma exposure and put gamma exposure across strikes and expiries.
  - **Positive Net GEX**: Dealers are modeled long gamma. To remain delta-neutral, dealers buy into market declines and sell into market rallies, exerting a volatility-dampening, mean-reverting effect.
  - **Negative Net GEX**: Dealers are modeled short gamma. Dealers must sell into falling prices and buy into rising prices to hedge, accelerating spot moves and amplifying volatility.
- **Gamma Flip**: The spot price threshold where dealer aggregate positioning crosses between positive and negative Net GEX.
- **Call Wall**: The strike price with the largest absolute Call Gamma concentration. Acts as a key dealer hedging sensitivity level where hedging activity shifts dramatically.
- **Put Wall**: The strike price with the largest absolute Put Gamma concentration (or primary downside gamma concentration), representing a critical dealer hedging pivot.
- **Max Pain**: The underlying strike price at which the aggregate cash payoff to option holders is minimized and option sellers retain maximum collective premium at expiration.

---

### Flow & Order Book Mechanics

- **Options Flow**: Normalized taker executions and block trades aggregated across time windows (1h, 4h, 24h), categorized by premium turnover, aggression side (buy/sell), contract type (call/put), and trade structure.
- **OI (Open Interest)**: The total quantity of outstanding, unliquidated option contracts active in the clearing house for a given instrument.
- **Dealer Hedging**: The systematic, algorithmic rebalancing of spot or perpetual futures executed by liquidity providers to maintain a delta-neutral book as spot prices and contract Greeks fluctuate.
- **Expiry**: The designated settlement date and time (typically 08:00 UTC for Deribit crypto options) when options exercise or expire.
- **DTE (Days to Expiration)**: Calendar days remaining until expiration and final cash or physical settlement.
