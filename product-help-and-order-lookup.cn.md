# Product Help and Order Lookup（工具与功能使用咨询）

## Description

**任务说明**：产品身份（如 Clawbot）、端侧访问方式、某功能在哪里打开、网格/订单状态「怎么用」等**说明型**意图；Skills 仅在涉及**真实查单、查行情**时介入，其余依赖官方文档与 Agent 通用知识。

**典型线上意图**：Clawbot 身份与功能；Binance AI iOS 访问；网格策略与订单状态查询；ETF 对价格影响（部分与 `market-data-and-analysis.cn.md` 重叠）。

---

## 推荐 Skills 组合

| 场景 | Skill | 用途 |
|------|--------|------|
| 查订单是否生效 | `spot` / `algo` / 对应 `derivatives-trading-*` | 按市场类型选唯一 Skill |
| 同上（**CLI**） | `binance` | `binance-cli spot` / `futures-usds` 查单与 REST 对照（见 `trading-execution.cn.md` §C） |
| 查行情是否支持 | `query-token-info` | 验证交易对与数据可用性 |
| 扩展技能 | `skill-creator` | 用户想自定义 Agent Skill 时的编写与优化 |
| 安全扩展 | `skill-vetter` | 安装第三方 Skill 前审查 |

---

## Plan（执行计划）

> 与 `Task_upgrade_advice.cn.md` §7 对齐：**先 FAQ 后接口**；仅「查我的单/网格」时进入读单流水线；先确认现货/合约/算法。

### 状态确认与不支持时的沟通

- **规划前需确认**：涉及「查我的单」时，确认 **现货/合约/算法**；若有 `orderId` 或 `origClientOrderId` 优先使用。
- **若查无订单或与用户描述的网格不符**：① 说明查询范围与结果；② 追问：是否其它账户、是否已成交/撤单、是否提供订单号/交易对/时间范围；③ 在确认市场类型前**不混用**不同 Skill 的 `openOrders`。
- **跨 Task 原则**：见 [Task_upgrade_advice.cn.md](./Task_upgrade_advice.cn.md) 开篇。

### A. 结构化流水线（DAG）

**Step 0: 前置状态诊断 (Prerequisite State Check) - *MANDATORY***
> **目标**: 在执行任何具体任务前，必须先全面了解用户的当前状态，以提供个性化、避免风险的建议。
> **核心 Skills**: `assets`, `spot` (for `getOrders`), `derivatives-trading-usds-futures` (for `getPositions`)

1.  **账户资产查询**: 调用 `assets.getUserAssets`，检查各钱包（特别是现货 `SPOT` 和资金 `FUNDING`）的可用余额、总估值。
2.  **当前持仓分析**: 调用 `derivatives-trading-usds-futures.getPositions`，检查用户是否有U本位合约持仓，了解其方向、大小和未实现盈亏。
3.  **历史交易与挂单**: 调用 `spot.getOrders`，检查用户近期的交易习惯（如偏好的币对）和当前有无未成交的挂单。

> **诊断后决策**: 根据诊断结果动态调整后续步骤。例如，如果用户已有相关持仓，应优先围绕该持仓展开计划；如果资金不足，则参考 `fuzzy-intent-and-account-onboarding.cn.md` 进行入金引导。

---


| 步骤 | 动作 |
|------|------|
| **分流** | 纯产品说明 → 无 API；涉及订单 → 确认 **现货 / 合约 U 本位 / 币本位 / 算法单**。 |
| **现货** | `openOrders`（按 symbol）→ 若无，`order` 或 `allOrders`（时间窗）。 |
| **算法** | `algo/spot/openOrders` 或 `algo/futures/openOrders`，与网格描述对齐。 |
| **合约** | `fapi` 或 `dapi` 的 `openOrders` + `positionRisk`。 |
| **币种可查** | `query-token-info` 的 `search` 验证有数据即可。 |

### B. 接口级速查

1. **演示「订单是否在」— 现货**
   - `GET https://api.binance.com/api/v3/openOrders`（签名的 USER_DATA）
   - `GET /api/v3/order`：`symbol` + `orderId` 或 `origClientOrderId`

2. **演示「算法单 / TWAP」**
   - `GET https://api.binance.com/sapi/v1/algo/spot/openOrders` 或 `.../algo/futures/openOrders`

3. **演示「合约订单/持仓」**
   - USDS-M：`GET /fapi/v1/openOrders`、`GET /fapi/v2/positionRisk`
   - COIN-M：`GET /dapi/v1/openOrders`

4. **验证某代币是否有 Web3 行情**
   - `GET https://web3.binance.com/bapi/defi/v5/public/wallet-direct/buw/wallet/market/token/search/ai?keyword=...`

5. **`skill-creator` / `skill-vetter`**：按 Cursor 侧 Skill 文档操作（非 Binance REST）；用于用户自建 Skill 或审查第三方包。

### C. binance-cli（可选，与 §B 对照）

已安装 **`binance-cli`** 时，可用 **`binance` Skill**（`skills/binance/binance/`）只读核对订单与持仓，与 §B REST **等价路径**：

- 现货：`binance-cli spot get-open-orders`、`binance-cli spot get-order`（参数见 `references/spot.md`）。
- USDS-M：`binance-cli futures-usds current-all-open-orders`、`binance-cli futures-usds query-order`（见 `references/futures-usds.md`）。

COIN-M、算法单、`algo` 等 **无** `binance-cli` 覆盖时仍用 §B REST。

---

## 使用指南

### 结构化要点

- **默认零请求**：能一句话说清的产品问题不要调 API。
- **市场未确认不调单**：避免在未知现货/合约的情况下盲打 `openOrders`。

### 接口约定

- **查单与下单同一 Skill**：现货仅使用 `spot` 的 `/api/v3/*`，不要用 `convert` 查现货限价单。
- **与 `trading-execution.cn.md`**：网格/止盈止损状态以对应市场的 `openOrders`、`allOrders` 或 `algo/*/openOrders` 为准。
