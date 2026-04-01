# Trading Execution

## Description

**Task**: Spot / Convert / Algo / Margin / Futures / Options “place, amend, cancel, TP/SL, grid, automation”; includes **data-side** support for strategies (quotes, order status) and Skill selection on the execution path.

**Typical intents**: quant / short-term strategies; auto trading and clearing orders; BNB futures entries and strategies; TP/SL; grid and order monitoring; DOGE/FDUSD grids; order status; BTC long execution, etc.

**Skill boundaries**: “Investment advice” must be compliant; actual orders must use the Skill that matches the instrument (spot ≠ futures ≠ options).

---

## Recommended Skills

| Scenario | Primary | Secondary |
|----------|-----------|-----------|
| Spot trade, cancel, query | `spot` | `convert` (quick swap) |
| Convert | `convert` | `query-token-info` (pair check) |
| TWAP etc. algo | `algo` | `spot` or matching derivatives |
| USDS-M perp/delivery | `derivatives-trading-usds-futures` | `algo` if supported |
| COIN-M perp/delivery | `derivatives-trading-coin-futures` | same |
| Options | `derivatives-trading-options` | `query-token-info` |
| Unified margin | `derivatives-trading-portfolio-margin` / `derivatives-trading-portfolio-margin-pro` | `assets` |
| Cross / isolated margin | `margin-trading` | `spot` |
| P2P quotes and orders | `p2p` | — |

---

## Plan

> Aligned with `Task_upgrade_advice.en.md` §3: **single market lock** → read-only orders/positions → precision & rules before writes → algo sub-order tracking; quotes from `market-data-and-analysis.en.md`, not order APIs as substitutes.

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Market lock** | Map to one Skill: `spot` / `convert` / `algo` / `fapi` / `dapi` / `eapi` / `papi` / `margin` / `p2p` (quotes only); **do not** mix `fapi` and `dapi` in one flow. |
| **Read-only** | `openOrders` → `order` (by id) → futures then `positionRisk`; confirm before cancel/amend. |
| **Write** | Spot: `exchangeInfo`/`myFilters` then `order`; futures: `leverage`/`marginType` if needed; convert: `exchangeInfo` → `getQuote` → confirm → `acceptQuote`. |
| **Algo** | After place: `algo/*/openOrders` + `subOrders` for child progress. |
| **Quotes** | Last/tranks from `market-data-and-analysis.en.md`. |

### B. API quick reference

**Bases**: Spot/Convert/Algo/Margin/SAPI → `https://api.binance.com`; USDS-M → `https://fapi.binance.com`; COIN-M → `https://dapi.binance.com`; Options → `https://eapi.binance.com` (per SKILL).

1. **Spot (`spot`)**
   - Order: `POST /api/v3/order` (`symbol`, `side`, `type`, `quantity`/`quoteOrderQty`, `price`, `timeInForce`, …).
   - Cancel: `DELETE /api/v3/order`; all: `DELETE /api/v3/openOrders`.
   - Query: `GET /api/v3/order`, `GET /api/v3/openOrders`, `GET /api/v3/allOrders`.
   - Conditional/OCO: see SKILL `/api/v3/orderList/*`, `/api/v3/order/oco`, etc.

2. **Convert (`convert`)**
   - Pairs: `GET /sapi/v1/convert/exchangeInfo`.
   - Quote: `POST /sapi/v1/convert/getQuote` (`fromAsset`, `toAsset`, `fromAmount`/`toAmount`).
   - Accept: `POST /sapi/v1/convert/acceptQuote` (`quoteId`).
   - Limit: `POST /sapi/v1/convert/limit/placeOrder`; status: `GET /sapi/v1/convert/orderStatus`, `GET /sapi/v1/convert/limit/queryOpenOrders`.

3. **Algo TWAP/VP (`algo`)**
   - Futures: `POST /sapi/v1/algo/futures/newOrderTwap`, `POST /sapi/v1/algo/futures/newOrderVp`; open: `GET /sapi/v1/algo/futures/openOrders`; sub: `GET /sapi/v1/algo/futures/subOrders?algoId=`.
   - Spot TWAP: `POST /sapi/v1/algo/spot/newOrderTwap`; open: `GET /sapi/v1/algo/spot/openOrders`.

4. **USDS-M (`derivatives-trading-usds-futures`)**
   - Order: `POST /fapi/v1/order`; cancel: `DELETE /fapi/v1/order`; batch: `POST /fapi/v1/batchOrders`.
   - Positions: `GET /fapi/v2/positionRisk`; balance: `GET /fapi/v2/balance`.
   - Conditional/algo: `POST /fapi/v1/algoOrder`, `GET /fapi/v1/openAlgoOrders`.

5. **COIN-M (`derivatives-trading-coin-futures`)**
   - `POST /dapi/v1/order`, `GET /dapi/v1/openOrders`, `GET /dapi/v1/positionRisk` (often `pair`/`symbol`).

6. **Options (`derivatives-trading-options`)**
   - `POST /eapi/v1/order`, `GET /eapi/v1/openOrders`, `GET /eapi/v1/position`, `DELETE /eapi/v1/allOpenOrders`.

7. **Portfolio margin (`derivatives-trading-portfolio-margin`)**
   - Account & orders: `GET /papi/v1/account`, `GET /papi/v2/um/account`, `POST /papi/v1/um/order` / `POST /papi/v1/cm/order`, etc. (see SKILL Quick Reference).

8. **Margin spot (`margin-trading`)**
   - `POST /sapi/v1/margin/order` (`isIsolated`, `sideEffectType`, …); query: `GET /sapi/v1/margin/openOrders`.

9. **P2P quotes & history (`p2p`)**
   - Public: `GET https://www.binance.com/bapi/c2c/v1/public/c2c/agent/quote-price` (`fiat`, `asset`, `tradeType`); `.../ad-list`; `.../trade-methods`.
   - Private: `GET /sapi/v1/c2c/orderMatch/listUserOrderHistory`, `GET /sapi/v1/c2c/orderMatch/getUserOrderSummary` (signed; **do not sort SAPI params**).

---

## Usage

### Structured notes

- **One flow per intent**: switching market = re-pick Skill.
- **Read before write**: except instant convert, confirm open orders/positions before changes.

### Conventions

- **`newClientOrderId` (spot)**: SKILL may require `agent-` prefix (auto if omitted).
- **USDS vs COIN**: do not mix `fapi` and `dapi`; options use `eapi`.
- **convert vs spot**: one-shot quote+accept → `convert/getQuote` + `acceptQuote`; limit book depth → `spot`.
- **With `market-data-and-analysis.en.md`**: on-chain / Web3 BAPI in that doc; this Task is exchange order APIs.
