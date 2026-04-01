# Campaigns and Rewards（活动与激励）

## Description

**任务说明**：平台活动、促销、Beta 激励、活动结果讨论等；Skills **不保证**包含实时活动页全文，可用 `alpha` 等与活动相关的代币能力作补充，其余依赖官网、公告与 Agent 知识库时效性。

**典型线上意图**：币安近期活动与促销；Beta 测试激励；活动结果与命名创意等社交向内容。

---

## 推荐 Skills 组合

| 角色 | Skill | 用途 |
|------|--------|------|
| 场景补充 | `alpha` | Alpha 相关代币与活动语境（若适用） |
| 通用 | `query-token-info` / `crypto-market-rank` | 活动涉及特定币时可拉行情 |
| 边界 | — | 活动规则以官方页面为准 |

---

## Plan（执行计划）

> 与 `Task_upgrade_advice.md` §9 对齐：**规则以官网为准** → 可选「活动提到的币」行情佐证 → 可选用户自查交易量；讨论类无 API。

### A. 结构化流水线（DAG）

| 步骤 | 动作 |
|------|------|
| **默认** | 活动/Beta 条款 → 文本 + 引导 App/官网；**无**独立活动全文 API。 |
| **可选：代币语境** | 活动点名某币：`alpha` token list 或 `unified/rank`（`rankType=20`）或 `search/ai`，仅证明「产品侧有该币语境」。 |
| **可选：任务进度** | 用户自查交易量：`getUserAsset` + `openOrders`/`allOrders`（时间窗）→ **不保证**与活动统计口径一致。 |
| **讨论** | 命名创意等 → 零 REST。 |

### B. 接口级速查

1. **声明**：活动/Beta 规则**无**独立活动 API 在本 Skill 集中；以官网与 App 为准。

2. **若活动绑定具体代币行情**（可选）
   - `alpha`：`GET https://www.binance.com/bapi/defi/v1/public/wallet-direct/buw/wallet/cex/alpha/all/token/list`
   - Web3：`POST .../unified/rank/list/ai`（`rankType=20` Alpha 榜）或 `query-token-info` 的 `search/ai`。

3. **若活动涉及交易任务**
   - 持仓/余额：`assets` 的 `POST /sapi/v3/asset/getUserAsset`；订单：见 `trading-execution.md` 中 `/api/v3/order`、`/fapi/v1/openOrders` 等。

4. **讨论类**：无需 REST。

---

## 使用指南

### 结构化要点

- **接口不能替代活动条款**：行情只能佐证价格/是否在榜，不能推导奖励资格。
- **默认零请求**：先回答规则与链接，再决定是否拉行情。

### 边界

- **轻量调用**：多数场景无需 API。
