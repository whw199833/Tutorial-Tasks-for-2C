# On-chain Signals and Security

## Description

**Task**: BSC-style transfer ideas, smart-money signals, token security audit, address holdings; and for “monitoring scripts, scheduled pushes, conditional alerts”—**data and signals** Skills can provide (long-running deployment still needs user env: cron / cloud functions).

**Typical intents**: BSC transfer lookup; smart-money monitoring and alerts; scheduled monitoring; buy signals; token audit; smart-money signals and token analysis.

---

## Recommended Skills

| Role | Skill | Use |
|------|--------|-----|
| Primary | `trading-signal` | Smart-money buy/sell signals |
| Primary | `query-token-audit` | Honeypot, rug risk, etc. |
| Primary | `query-address-info` | On-chain address PnL / positions |
| Secondary | `query-token-info` | Token basics and price context |
| Secondary | `meme-rush` | Meme sector heat |

---

## Plan

> Aligned with `Task_upgrade_advice.en.md` §5: **audit first if contract**; then signals; for signal tokens use `dynamic` for liquidity; address monitoring is paginated; Meme can align narrative.

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Audit first** | If `contractAddress` exists: **`query-token-audit` first**; high risk / honeypot → **risk first**, then decide whether to expand signals. |
| **Smart money** | `trading-signal` → sort/filter by `direction`, `maxGain`, `exitRate`, `alertPrice` vs `currentPrice`. |
| **Deepen** | For selected contracts: `dynamic/info` liquidity; if very low, soften “actionable” wording. |
| **Address** | `query-address-info` paginated; “continuous monitor” = user-side **polling** same API (cron); Skill does not host daemons. |
| **Meme** | New launch talk: `meme-rush` stage vs signal list same narrative or not. |

### B. API quick reference

1. **Token audit (`query-token-audit`)** (call first when contract known)
   - `POST https://web3.binance.com/bapi/defi/v1/public/wallet-direct/security/token/audit`
   - JSON: `binanceChainId` (`56`/`8453`/`CT_501`/`1`), `contractAddress`, `requestId` (UUID v4); optional header `source: agent`.

2. **Smart-money (`trading-signal`)**
   - `POST https://web3.binance.com/bapi/defi/v1/public/wallet-direct/buw/wallet/web/signal/smart-money/ai`
   - Body: `chainId` (`56` BSC, `CT_501` Solana), `page`, `pageSize`; response includes `alertPrice`, `currentPrice`, `maxGain`, `exitRate`, `direction`, etc.

3. **Address positions (`query-address-info`)**
   - `GET https://web3.binance.com/bapi/defi/v3/public/wallet-direct/buw/wallet/address/pnl/active-position-list/ai`
   - Query: `address`, `chainId`, `offset`; Headers: `clienttype: web`, `clientversion: 1.2.0`.

4. **Quote context (`query-token-info`)**
   - `GET .../market/token/search/ai` or `.../token/dynamic/info/ai` (see `market-data-and-analysis.en.md`) for signal token price and liquidity.

5. **Meme (`meme-rush`)**
   - `POST .../buw/wallet/market/token/pulse/rank/list/ai` (`rankType` 10/20/30); Topic: `GET .../social-rush/rank/list/ai`.

6. **Monitoring**: all **pull REST**; scheduled push = user cron polling same endpoints; no WebSocket deployment in Skill.

---

## Usage

### Structured notes

- **Audit ≠ tradable**: pass still means technical scan only, not return or endorsement.
- **Signals + liquidity**: avoid over-interpreting smart-money on ultra-thin liquidity.

### Chains & privacy

- **Chain IDs**: BSC `56`, Base `8453`, Solana `CT_501`, Ethereum `1` (audit).
- **With `market-data-and-analysis.en.md`**: ranks and mood there; this Task is address- and signal-level.
- **Privacy**: redact wallet addresses / positions in UI when appropriate.
