# Trading Execution

## Overview

| API | Function | Use Case |
|-----|----------|----------|
| Spot / convert / algo | REST `/api/v3`, SAPI convert, algo | Order lifecycle per cash/algo market |
| USDS-M / COIN-M / options | `fapi` / `dapi` / `eapi` | Futures and options execution |
| Portfolio margin | PM REST (`papi`) | Unified margin flows when in scope |
| `binance-cli` | `binance` skill (spot / convert / futures-usds) | CLI cross-check where enabled |

## Description

**Task summary**: Spot / convert / algo / margin / futures / options: place, amend/cancel, query orders, take-profit/stop-loss, grids, automated trading; **data-side** support for strategy advice (market data, order state) and skill selection on the execution path.

**Typical intents**: Quant/short-term strategies; auto-trading and clearing orders; BNB futures entries and strategy; TP/SL; grid trading and order monitoring; DOGE/FDUSD grids; order status; long BTC execution, etc.

**Skill boundaries**: “Investment advice” must stay compliant; real orders must use the skill for the **actual product** (spot ≠ futures ≠ options). **Execution path**: same market via **REST** (per-product `SKILL.md` quick reference) or, when `binance-cli` is installed, **`binance`** skill (`spot` / `convert` / `futures-usds`, cross-checked with spot, convert, USDS-M REST).

---

## Recommended skill mix

| Scenario | Primary skill | Secondary |
|----------|---------------|-----------|
| Spot / convert / USDS-M (**CLI**) | `binance` (`binance-cli`) | Cross-check with `spot` / `convert` / `derivatives-trading-usds-futures` |
| Spot trade, cancel, query | `spot` | `convert` (quick swap) |
| Convert | `convert` | `query-token-info` (pair check) |
| TWAP etc. algo | `algo` | `spot` or relevant derivative market |
| USDS-M perp/delivery | `derivatives-trading-usds-futures` | `algo` if supported |
| COIN-M perp/delivery | `derivatives-trading-coin-futures` | same |
| Options | `derivatives-trading-options` | `query-token-info` |
| Unified portfolio margin | `derivatives-trading-portfolio-margin` / `derivatives-trading-portfolio-margin-pro` | `assets` |
| Cross/isolated margin spot | `margin-trading` | `spot` |
| P2P quotes and history | `p2p` | — |

---

## Plan

**Step 0: Prerequisite state check — *MANDATORY***

> **Goal**: Understand full user state before actions.  
> **Core skills**: `assets`, `spot`, `derivatives-trading-usds-futures`

1. **Assets**: `assets.getUserAssets` (`SPOT`, `FUNDING`).
2. **Positions**: `derivatives-trading-usds-futures.getPositions`.
3. **Orders**: `spot.getOrders` for habits and open orders.

> **After diagnosis**: If underfunded, **[fuzzy-intent-and-account-onboarding.md](./fuzzy-intent-and-account-onboarding.md)**.

---

> Aligns with `task-upgrade-advice.md` §3: **single market lock** → read-only orders/positions → precision/rules before writes → algo sub-order tracking; prices from [market-data-and-analysis.md](./market-data-and-analysis.md)—do not use order APIs as price feeds.

### Status checks and when you cannot proceed

- **Before writes**: **Single market** locked; **available balance** and **margin**; **open orders and positions** (incl. algo, grids); avoid in-flight conflicts.
- **If under-margined or missing params**: (1) State gap and ceiling; (2) Ask about deposit/transfer, cancel/reduce, smaller size or leverage; (3) **Do not** place orders or confirm convert for the user until confirmed.
- **Cross-task rules**: [task-upgrade-advice.md](./task-upgrade-advice.md).

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Market lock** | One skill: `spot` / `convert` / `algo` / `fapi` / `dapi` / `eapi` / `papi` / `margin` / `p2p` (quotes only); **never** mix `fapi` and `dapi` in one flow. CLI: `binance-cli spot` \| `convert` \| `futures-usds` (see §C). |
| **Read-only** | `openOrders` → `order` (by id) → futures `positionRisk`; CLI e.g. `get-open-orders`, `query-order`, `position-information-v-3`; cancel/amend after confirm. |
| **Writes** | Spot: `exchangeInfo`/`myFilters` then `order`; futures: `leverage`/`marginType` if needed; convert: `exchangeInfo` → `getQuote` → confirm → `acceptQuote`. CLI: §C; **prod requires user `CONFIRM`**. |
| **Algo** | After place: `algo/*/openOrders` + `subOrders`. |
| **Market data** | From [market-data-and-analysis.md](./market-data-and-analysis.md). |

### B. REST quick reference

**Bases**: Spot/Convert/Algo/Margin/SAPI → `https://api.binance.com`; USDS-M → `https://fapi.binance.com`; COIN-M → `https://dapi.binance.com`; Options → `https://eapi.binance.com`.

1. **Spot (`spot`)**: `POST /api/v3/order`; `DELETE /api/v3/order`; `DELETE /api/v3/openOrders`; `GET /api/v3/order`, `openOrders`, `allOrders`; conditional/OCO per SKILL.
2. **Convert (`convert`)**: `GET /sapi/v1/convert/exchangeInfo`; `POST .../getQuote`; `POST .../acceptQuote`; limit orders per SKILL.
3. **Algo TWAP/VP (`algo`)**: futures/spot TWAP and VP endpoints per SKILL.
4. **USDS-M**: `POST /fapi/v1/order`; `DELETE /fapi/v1/order`; `POST /fapi/v1/batchOrders`; `GET /fapi/v2/positionRisk`; `GET /fapi/v2/balance`; algo orders per SKILL.
5. **COIN-M**: `POST /dapi/v1/order`, `GET /dapi/v1/openOrders`, `GET /dapi/v1/positionRisk`, etc.
6. **Options**: `POST /eapi/v1/order`, `GET /eapi/v1/openOrders`, `GET /eapi/v1/position`, `DELETE /eapi/v1/allOpenOrders`.
7. **PM**: `GET /papi/v1/account`, UM/CM order endpoints per SKILL.
8. **Margin spot**: `POST /sapi/v1/margin/order`; `GET /sapi/v1/margin/openOrders`.
9. **P2P**: public `quote-price`, `ad-list`, `trade-methods`; private order history—**do not sort** SAPI params.

### C. `binance-cli` (`binance` skill, vs §B)

| Subcommand | REST skill | Hub ref |
|------------|------------|---------|
| `binance-cli spot <endpoint>` | `spot` | [`spot.md`](../../../binance/binance/references/spot.md) |
| `binance-cli convert <endpoint>` | `convert` | [`convert.md`](../../../binance/binance/references/convert.md) |
| `binance-cli futures-usds <endpoint>` | `derivatives-trading-usds-futures` | [`futures-usds.md`](../../../binance/binance/references/futures-usds.md) |

Install via `@binance/binance-cli`; **`CONFIRM`** for prod; `algo`, `p2p`, margin, COIN-M, options, PM → REST only.

---

## Usage guide

- **One intent, one market chain**; changing market = re-pick skill.
- **Read before write** except convert instant fill.
- **`newClientOrderId`**: `agent-` prefix per spot SKILL.
- **USDS vs COIN**: no `fapi`/`dapi` mix; options use `eapi`.
- **convert vs spot**: one-tap quote+fill vs limit book depth.
- **With [market-data-and-analysis.md](./market-data-and-analysis.md)**: on-chain/BAPI there; this task is exchange order APIs.
- **CLI vs REST**: no double-submit on same trade (read-only cross-check OK).
