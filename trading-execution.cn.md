# Trading Execution（交易策略与执行）

## Description

**任务说明**：覆盖现货/闪兑/算法单/杠杆/合约/期权等「下单、改单、查单、止盈止损、网格与自动化交易」类意图；包括策略建议的**数据侧支撑**（行情、订单状态），以及执行路径上的 Skill 选型。

**典型线上意图**：量化/短线策略；自动交易与清理挂单；BNB 期货开仓与策略；止盈止损；网格交易与订单监控；DOGE/FDUSD 网格；查询订单状态；BTC 多头执行等。

**Skills 边界**：「投资建议」需合规表述；实际下单必须选对交易品种对应的 Skill（现货 ≠ 合约 ≠ 期权）。

---

## 推荐 Skills 组合

| 场景 | 主 Skill | 辅助 |
|------|-----------|------|
| 现货买卖、撤单、查单 | `spot` | `convert`（快速换币） |
| 闪兑 | `convert` | `query-token-info`（核对交易对） |
| TWAP 等算法单 | `algo` | `spot` 或衍生品下对应市场 |
| USDS-M 永续/交割 | `derivatives-trading-usds-futures` | `algo`（若支持该市场） |
| COIN-M 永续/交割 | `derivatives-trading-coin-futures` | 同上 |
| 期权 | `derivatives-trading-options` | `query-token-info` |
| 统一账户保证金 | `derivatives-trading-portfolio-margin` / `derivatives-trading-portfolio-margin-pro` | `assets` |
| 全仓/逐仓杠杆 | `margin-trading` | `spot` |
| P2P 行情与订单查询 | `p2p` | — |

---

## Plan（执行计划）

> 与 `Task_upgrade_advice.cn.md` §3 对齐：**唯一市场锁定** → 只读查单/持仓 → 写单前精度与规则 → 算法单追踪；行情来自 `market-data-and-analysis.cn.md`，不用订单接口代替。

### A. 结构化流水线（DAG）

| 步骤 | 动作 |
|------|------|
| **市场锁定** | 映射到单一 Skill：`spot` / `convert` / `algo` / `fapi` / `dapi` / `eapi` / `papi` / `margin` / `p2p`（仅行情）；**禁止**同流程混用 `fapi` 与 `dapi`。 |
| **只读** | `openOrders` → `order`（按 id）→ 合约再 `positionRisk`；确认后再撤/改。 |
| **写单** | 现货：`exchangeInfo`/`myFilters` 再 `order`；合约：必要时先 `leverage`/`marginType`；闪兑：`exchangeInfo` → `getQuote` → 确认 → `acceptQuote`。 |
| **算法单** | 下单后 `algo/*/openOrders` + `subOrders` 跟子单进度。 |
| **行情** | 现价/榜单从 `market-data-and-analysis.cn.md` 拉取。 |

### B. 接口级速查

**Base**：Spot/Convert/Algo/Margin/SAPI → `https://api.binance.com`；USDS-M → `https://fapi.binance.com`（文档与 SKILL 一致）；COIN-M → `https://dapi.binance.com`；Options → `https://eapi.binance.com`（以 SKILL 为准）。

1. **现货（`spot`）**
   - 下单：`POST /api/v3/order`（`symbol`、`side`、`type`、`quantity`/`quoteOrderQty`、`price`、`timeInForce` 等）。
   - 撤单：`DELETE /api/v3/order`；全撤：`DELETE /api/v3/openOrders`。
   - 查单：`GET /api/v3/order`、`GET /api/v3/openOrders`、`GET /api/v3/allOrders`。
   - 条件/OCO 组合：见 SKILL 中 `/api/v3/orderList/*`、`/api/v3/order/oco` 等。

2. **闪兑（`convert`）**
   - 交易对：`GET /sapi/v1/convert/exchangeInfo`。
   - 询价：`POST /sapi/v1/convert/getQuote`（`fromAsset`、`toAsset`、`fromAmount`/`toAmount`）。
   - 成交：`POST /sapi/v1/convert/acceptQuote`（`quoteId`）。
   - 限价单：`POST /sapi/v1/convert/limit/placeOrder`；查询：`GET /sapi/v1/convert/orderStatus`、`GET /sapi/v1/convert/limit/queryOpenOrders`。

3. **算法单 TWAP/VP（`algo`）**
   - 合约：`POST /sapi/v1/algo/futures/newOrderTwap`、`POST /sapi/v1/algo/futures/newOrderVp`；查询开放单：`GET /sapi/v1/algo/futures/openOrders`；子单：`GET /sapi/v1/algo/futures/subOrders?algoId=`。
   - 现货 TWAP：`POST /sapi/v1/algo/spot/newOrderTwap`；查询：`GET /sapi/v1/algo/spot/openOrders`。

4. **USDS-M 合约（`derivatives-trading-usds-futures`）**
   - 下单：`POST /fapi/v1/order`；撤单：`DELETE /fapi/v1/order`；批量：`POST /fapi/v1/batchOrders`。
   - 持仓：`GET /fapi/v2/positionRisk`；余额：`GET /fapi/v2/balance`。
   - 条件/算法单：`POST /fapi/v1/algoOrder`，`GET /fapi/v1/openAlgoOrders`。

5. **COIN-M 合约（`derivatives-trading-coin-futures`）**
   - `POST /dapi/v1/order`、`GET /dapi/v1/openOrders`、`GET /dapi/v1/positionRisk` 等（参数常含 `pair`/`symbol`）。

6. **期权（`derivatives-trading-options`）**
   - `POST /eapi/v1/order`、`GET /eapi/v1/openOrders`、`GET /eapi/v1/position`、`DELETE /eapi/v1/allOpenOrders`。

7. **统一账户 PM（`derivatives-trading-portfolio-margin`）**
   - 账户与下单：`GET /papi/v1/account`、`GET /papi/v2/um/account`、`POST /papi/v1/um/order` / `POST /papi/v1/cm/order` 等（见 SKILL Quick Reference）。

8. **杠杆现货（`margin-trading`）**
   - `POST /sapi/v1/margin/order`（`isIsolated`、`sideEffectType` 等）；查单：`GET /sapi/v1/margin/openOrders`。

9. **P2P 仅行情与历史（`p2p`）**
   - 公开：`GET https://www.binance.com/bapi/c2c/v1/public/c2c/agent/quote-price`（`fiat`、`asset`、`tradeType`）；`.../ad-list`；`.../trade-methods`。
   - 私有：`GET /sapi/v1/c2c/orderMatch/listUserOrderHistory`、`GET /sapi/v1/c2c/orderMatch/getUserOrderSummary`（需签名；**SAPI 参数勿排序**）。

---

## 使用指南

### 结构化要点

- **一单一流**：每个用户意图只走一条市场链路；切换市场等于重新选 Skill。
- **先读后写**：除闪兑瞬时成交外，优先确认挂单/持仓再改单。

### 接口与约定

- **`newClientOrderId`（spot）**：Skill 要求必须以 `agent-` 为前缀（若未传则自动生成带前缀）。
- **USDS vs COIN**：`fapi` 与 `dapi` 不可混用；期权用 `eapi`。
- **convert vs spot**：一键询价成交走 `convert/getQuote` + `acceptQuote`；挂单深度用 `spot`。
- **与 `market-data-and-analysis.cn.md`**：链上/行情用 Web3 BAPI（见该文档）；本 Task 为交易所订单 API。
