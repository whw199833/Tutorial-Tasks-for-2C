# Product Help and Order Lookup

## Description

**Task**: Product identity (e.g. Clawbot), how to access features, where to open a feature, grid/order status “how to”—**explanatory** intents; Skills only when **real order or quote** lookup is needed; otherwise official docs and general Agent knowledge.

**Typical intents**: Clawbot identity and features; Binance AI on iOS; grid strategy and order status; ETF price impact (partly overlaps `market-data-and-analysis.en.md`).

---

## Recommended Skills

| Scenario | Skill | Use |
|----------|--------|-----|
| Check if order exists | `spot` / `algo` / matching `derivatives-trading-*` | One Skill per market |
| Check if symbol has data | `query-token-info` | Pair and data availability |
| Extend skills | `skill-creator` | User wants custom Agent Skills |
| Safety | `skill-vetter` | Before installing third-party Skills |

---

## Plan

> Aligned with `Task_upgrade_advice.en.md` §7: **FAQ first, APIs second**; read-only order pipeline only for “my orders / grid”; confirm spot/futures/algo first.

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Triage** | Pure product explanation → no API; if orders → confirm **spot / USDS-M / COIN-M / algo**. |
| **Spot** | `openOrders` (by `symbol`) → if empty, `order` or `allOrders` (time window). |
| **Algo** | `algo/spot/openOrders` or `algo/futures/openOrders`, aligned with grid description. |
| **Futures** | `fapi` or `dapi` `openOrders` + `positionRisk`. |
| **Symbol check** | `query-token-info` `search` to verify data exists. |

### B. API quick reference

1. **Demo “order exists”—spot**
   - `GET https://api.binance.com/api/v3/openOrders` (signed USER_DATA)
   - `GET /api/v3/order`: `symbol` + `orderId` or `origClientOrderId`

2. **Demo “algo / TWAP”**
   - `GET https://api.binance.com/sapi/v1/algo/spot/openOrders` or `.../algo/futures/openOrders`

3. **Demo “futures orders / positions”**
   - USDS-M: `GET /fapi/v1/openOrders`, `GET /fapi/v2/positionRisk`
   - COIN-M: `GET /dapi/v1/openOrders`

4. **Check Web3 quote for a token**
   - `GET https://web3.binance.com/bapi/defi/v5/public/wallet-direct/buw/wallet/market/token/search/ai?keyword=...`

5. **`skill-creator` / `skill-vetter`**: follow Cursor Skill docs (not Binance REST); for user-built or third-party Skill packages.

---

## Usage

### Structured notes

- **Default zero requests**: one-sentence product answers should not call APIs.
- **No blind orders**: do not call `openOrders` before market type is known.

### Conventions

- **Query orders with same Skill as trading**: spot orders only `spot` `/api/v3/*`, not `convert` for spot limit orders.
- **With `trading-execution.en.md`**: grid / TP-SL status uses that market’s `openOrders`, `allOrders`, or `algo/*/openOrders`.
