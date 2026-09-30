# GEX Monitor Agent Skills

Agent Skills for GEX Monitor — Crypto Options, Gamma, Volatility, Flow & Market Structure Research.

---

## Installation

Install the GEX Monitor skill into your AI agent environment via the official [Skills CLI](https://skills.sh):

```bash
npx skills add gexmonitor-pixel/gexmonitor-skills
```

To install explicitly specifying the skill name:

```bash
npx skills add gexmonitor-pixel/gexmonitor-skills --skill gexmonitor
```

Or using the full GitHub repository URL:

```bash
npx skills add https://github.com/gexmonitor-pixel/gexmonitor-skills --skill gexmonitor
```

---

## What It Gives Your Agent

This skill equips AI agents (including Claude Code, Codex, Cursor, Antigravity, and other compatible systems) with specialized methodology, tool routing, and multi-step research workflows for cryptocurrency derivatives:

- **Market Structure & Snapshots**: Real-time reference spot, Gamma regimes, and distance metrics for BTC, ETH, and SOL.
- **Gamma Exposure (GEX)**: Granular strike-level gamma distribution, Gamma Flip pivots, and Call/Put Wall identification.
- **Volatility Analytics**: ATM Implied Volatility (IV), Realized Volatility (RV), Volatility Risk Premium (VRP), and 25d/10d Risk Reversal Skew.
- **Options Flow**: Rolling taker volume, turnover, and buy/sell aggression across 1h, 4h, and 24h windows.
- **Whale Activity**: Institutional block trades, large prints, and inferred Open Interest (OI) positioning changes.
- **Expiry Attribution**: Per-expiry Gamma concentration, maturity weights, and roll/pin risk identification.
- **Option Strategy Analysis**: Quote-based deterministic economics, Greeks, and scenario payoffs for vertical spreads, straddles, and single legs.

---

## Live Data & Architecture

- **Skill (Methodology)**: This repository provides analytical workflows, routing logic, interpretation guidelines, and terminology definitions. It contains no static market data.
- **MCP (Live Data)**: Real-time, authenticated market-structure data is delivered via the GEX Monitor Remote MCP server:
  ```
  https://gexmonitor.com/api/mcp
  ```

Authentication is handled securely through GEX Monitor's OAuth 2.0 / dynamic client registration architecture.

---

## Example Prompts

### English
- *"Use GEX Monitor to analyze current BTC market structure."*
- *"Is BTC implied volatility currently expensive relative to realized volatility?"*
- *"Where are the critical Gamma levels and walls for ETH?"*
- *"What are whales doing in BTC options over the last 4 hours?"*
- *"How is today's BTC market structure different from yesterday's daily observation?"*
- *"Analyze a BTC 95k/105k bull call spread expiring next month using live quotes."*

### 中文
- *“使用 GEX Monitor 分析当前 BTC 的市场微观结构与 Gamma 分布。”*
- *“当前 ETH 的隐含波动率（IV）是否偏贵？VRP 水平如何？”*
- *“BTC 的 Gamma Flip 和主要 Call Wall / Put Wall 集中在哪些行权价？”*
- *“过去 24 小时内大户（Whale）在期权市场上有哪些大单异动和未平仓合约变化？”*
- *“评估这个 BTC 牛市看涨期权价差（Bull Call Spread）的盈亏平衡点与希腊字母敞口。”*

---

## Security & Privacy

- **No Secrets in Repository**: This skill repository contains purely declarative Markdown instructions and workflows. No API keys, database credentials, or private secrets are stored here.
- **OAuth Authentication**: Connecting agents authenticate directly against the GEX Monitor Remote MCP endpoint via OAuth. Users should never manually expose client secrets, database passwords, or bearer tokens.
- **Research Only**: GEX Monitor MCP tools are strictly read-only and analytical. No trading execution or capital movement capabilities exist.

---

## License

[MIT](LICENSE) © 2026 GEX Monitor
