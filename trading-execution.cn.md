# Trading Execution（交易策略与执行）

## Description

**任务说明**：覆盖现货/闪兑/算法单/杠杆/合约/期权等「下单、改单、查单、止盈止损、网格与自动化交易」类意图；包括策略建议的**数据侧支撑**（行情、订单状态），以及执行路径上的 Skill 选型。

**典型线上意图**：量化/短线策略；自动交易与清理挂单；BNB 期货开仓与策略；止盈止损；网格交易与订单监控；DOGE/FDUSD 网格；查询订单状态；BTC 多头执行等。

**Skills 边界**：「投资建议」需合规表述；实际下单必须选对交易品种对应的 Skill（现货 ≠ 合约 ≠ 期权）。**执行路径**：同一市场既可走 **REST**（各产品 `SKILL.md` 的 Quick Reference），也可在已安装 `binance-cli` 时走 **`binance` Skill**（`spot` / `convert` / `futures-usds` 子命令，与现货、闪兑、U 本位合约 REST 对照）。

---

## 推荐 Skills 组合

| 场景 | 主 Skill | 辅助 |
|------|-----------|------|
| 现货 / 闪兑 / U 本位合约（**CLI**） | `binance`（`binance-cli`） | 与下表 `spot` / `convert` / `derivatives-trading-usds-futures` **对照**，二选一或交叉核对 |
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

**Step 0: 前置状态诊断 (Prerequisite State Check) - *MANDATORY***
> **目标**: 在执行任何具体任务前，必须先全面了解用户的当前状态，以提供个性化、避免风险的建议。
> **核心 Skills**: `assets`, `spot` (for `getOrders`), `derivatives-trading-usds-futures` (for `getPositions`)

1.  **账户资产查询**: 调用 `assets.getUserAssets`，检查各钱包（特别是现货 `SPOT` 和资金 `FUNDING`）的可用余额、总估值。
2.  **当前持仓分析**: 调用 `derivatives-trading-usds-futures.getPositions`，检查用户是否有U本位合约持仓，了解其方向、大小和未实现盈亏。
3.  **历史交易与挂单**: 调用 `spot.getOrders`，检查用户近期的交易习惯（如偏好的币对）和当前有无未成交的挂单。

> **诊断后决策**: 根据诊断结果动态调整后续步骤。例如，如果用户已有相关持仓，应优先围绕该持仓展开计划；如果资金不足，则参考 `fuzzy-intent-and-account-onboarding.cn.md` 进行入金引导。

---

> 与 `Task_upgrade_advice.cn.md` §3 对齐：**唯一市场锁定** → 只读查单/持仓 → 写单前精度与规则 → 算法单追踪；行情来自 `market-data-and-analysis.cn.md`，不用订单接口代替。

### 状态确认与不支持时的沟通

- **规划前需确认**（写单/改单前必选）：**唯一市场**锁定；**可用余额**与**保证金**（衍生品/杠杆）；**现有挂单与持仓**（含算法单、网格）；避免与在途单冲突。
- **若资金不足、保证金不足或关键参数缺失**：① 说明缺口与可量化上限（如可买数量）；② 追问用户：是否充值/划转、是否先撤单或减仓、是否接受更小数量或调整杠杆；③ **不在未确认**前执行写单或代客确认闪兑。
- **跨 Task 原则**：见 [Task_upgrade_advice.cn.md](./Task_upgrade_advice.cn.md) 开篇。

### A. 结构化流水线（DAG）

| 步骤 | 动作 |
|------|------|
| **市场锁定** | 映射到单一 Skill：`spot` / `convert` / `algo` / `fapi` / `dapi` / `eapi` / `papi` / `margin` / `p2p`（仅行情）；**禁止**同流程混用 `fapi` 与 `dapi`。若用 CLI：`binance-cli spot` \| `convert` \| `futures-usds` 与上列一一对应（见 §C）。 |
| **只读** | `openOrders` → `order`（按 id）→ 合约再 `positionRisk`；CLI 侧如 `get-open-orders`、`query-order`、`position-information-v-3`；确认后再撤/改。 |
| **写单** | 现货：`exchangeInfo`/`myFilters` 再 `order`；合约：必要时先 `leverage`/`marginType`；闪兑：`exchangeInfo` → `getQuote` → 确认 → `acceptQuote`。CLI 见 §C；**prod 交易前须用户输入 `CONFIRM`**（`binance` SKILL）。 |
| **算法单** | 下单后 `algo/*/openOrders` + `subOrders` 跟子单进度。 |
| **行情** | 现价/榜单从 `market-data-and-analysis.cn.md` 拉取。 |

### B. 接口级速查（REST）

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

### C. binance-cli（`binance` Skill，与 §B 对照）

`binance-skills-hub/skills/binance/binance/SKILL.md`：**`binance-cli`**（依赖 `BINANCE_API_KEY`、`BINANCE_API_SECRET`；可选 `BINANCE_API_ENV`）。与 **§B 第 1、2、4 节**（现货、闪兑、USDS-M）常用操作 **REST 等价**，由 Agent 在 **REST Skill** 与 **CLI** 间二选一或交叉核对。

| 子命令 | 对应 REST Skill | Hub 参考 |
|--------|-----------------|----------|
| `binance-cli spot <endpoint>` | `spot` | `references/spot.md`（如 `get-account`、`get-open-orders`、`new-order`、`get-order`、`delete-order`） |
| `binance-cli convert <endpoint>` | `convert` | `references/convert.md`（如 `list-all-convert-pairs`、`send-quote-request`、`accept-quote`、`order-status`） |
| `binance-cli futures-usds <endpoint>` | `derivatives-trading-usds-futures` | `references/futures-usds.md`（如 `position-information-v-3`、`current-all-open-orders`、`new-order`、`query-order`） |

- **安装**：`metadata.openclaw.install` 指向 `@binance/binance-cli`（见 `binance` SKILL）；**Profile**：`binance-cli profile create` / `change`，任意命令可加 `--profile <name>`。
- **安全与约定**：生产环境成交类操作前请用户输入 **`CONFIRM`**；`--recvWindow`、Unix ms 时间戳等同 REST；细则见 `references/auth.md`。
- **CLI 未覆盖**：`algo`、`p2p`、`margin-trading`、COIN-M、期权、PM 等仍仅用 §B 对应 **REST** `SKILL.md`。

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
- **`binance` vs REST**：优先遵循部署环境已启用的能力；CLI 与 REST **不得**在同一笔操作上重复提交（除非一为只读核对）。
