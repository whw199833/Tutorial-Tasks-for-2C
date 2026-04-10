# Product Help and Order Lookup

## Overview

| API | Function | Use Case |
|-----|----------|----------|
| Spot / algo | `spot`, algo SAPI | Open orders, historical order, grid-style lookup |
| Futures | `derivatives-trading-*` REST | USDS-M / COIN-M positions and orders |
| `binance-cli` | `binance` skill | Optional read-only cross-check |
| Web3 symbol | `query-token-info` | Verify pair/token exists before lookups |

## Description

**Task summary**: Product identity (e.g. Clawbot), client access, where to open a feature, grid/order “how to”—**explanatory** intents; skills only when **real order or market lookups** are needed; else official docs and general agent knowledge.

**Typical intents**: Clawbot identity/features; Binance AI iOS access; grid strategy and order status; ETF price impact (overlaps [market-data-and-analysis.md](./market-data-and-analysis.md) partly).

---

## Recommended skill mix

| Scenario | Skill | Use |
|----------|--------|-----|
| Check if order exists | `spot` / `algo` / relevant `derivatives-trading-*` | One skill per market |
| Same (**CLI**) | `binance` | `binance-cli spot` / `futures-usds` vs REST ([trading-execution.md](./trading-execution.md) §C) |
| Check if market data exists | `query-token-info` | Pair/data availability |
| Extension | `skill-creator` | User-built agent skills |
| Security | `skill-vetter` | Third-party skill review |

---

## Plan

> Aligns with `task-upgrade-advice.md` §7: **FAQ before APIs**; only “my order/grid” → read-order flow; confirm spot/futures/algo first.

### Status checks and when you cannot proceed

- **Before planning**: For “my order”, confirm **spot / futures / algo**; prefer `orderId` or `origClientOrderId`.
- **If no order or grid mismatch**: (1) State query scope and result; (2) Ask other account?, filled/canceled?, id/symbol/time range?; (3) **Do not** mix `openOrders` across skills before market type is known.
- **Cross-task rules**: [task-upgrade-advice.md](./task-upgrade-advice.md).

### A. Structured pipeline (DAG)

**Step 0: Prerequisite state check — *MANDATORY*** when user ties help to account state.

---

| Step | Action |
|------|--------|
| **Triage** | Pure product Q → no API; orders → confirm **spot / USDS-M / COIN-M / algo**. |
| **Spot** | `openOrders` (by symbol) → if empty, `order` or `allOrders` (time window). |
| **Algo** | `algo/spot/openOrders` or `algo/futures/openOrders` vs grid description. |
| **Futures** | `fapi` or `dapi` `openOrders` + `positionRisk`. |
| **Symbol check** | `query-token-info` `search` to verify data exists. |

### B. Endpoint quick reference

1. **Spot open/order**: `GET /api/v3/openOrders`, `GET /api/v3/order`.
2. **Algo/TWAP**: `GET /sapi/v1/algo/spot/openOrders` or `.../algo/futures/openOrders`.
3. **Futures**: USDS-M `GET /fapi/v1/openOrders`, `GET /fapi/v2/positionRisk`; COIN-M `GET /dapi/v1/openOrders`.
4. **Web3 symbol**: `GET .../token/search/ai?keyword=...`
5. **`skill-creator` / `skill-vetter`**: per Cursor/skill docs—not Binance REST.

### C. `binance-cli` (optional)

Read-only cross-check: `binance-cli spot get-open-orders`, `get-order`; `binance-cli futures-usds current-all-open-orders`, `query-order`. COIN-M, algo without CLI → §B REST.

---

## Usage guide

- **Default zero calls**: one-sentence product answers need no API.
- **Unknown market → no blind `openOrders`**.
- **Lookup and trade same skill**: spot orders only `spot` `/api/v3/*`, not `convert` for spot limits.
- **With [trading-execution.md](./trading-execution.md)**: grid/TP-SL status via `openOrders`, `allOrders`, `algo/*/openOrders` for that market.
