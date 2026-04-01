# Simple Earn and VIP Loan（理财与借贷 · 简单赚币 / VIP 借贷）

## Description

**任务说明**：活期/定期申购赎回、简单赚币产品查询与操作，以及 VIP 借贷相关咨询与能力范围内的查询；与纯「现货余额」区分，侧重**生息与借贷**产品路径。

**典型用户表述**：申购赎回、年化、定期、借币利率、VIP 借贷额度与操作等（与 intent 分类中的账户资产可交叉，本 Task 强调产品类型）。

---

## 推荐 Skills 组合

| 角色 | Skill | 用途 |
|------|--------|------|
| 主路径 | `simple-earn` | 简单赚币（活期/定期等） |
| 主路径 | `vip-loan` | VIP 借贷服务 |
| 辅助 | `assets` | 资金划出/划入理财前后的钱包视图 |
| 辅助 | `margin-trading` | 用户混淆「借贷」与「杠杆借币」时澄清产品边界 |

---

## Plan（执行计划）

> 与 `Task_upgrade_advice.md` §12 对齐：**先定产品线**（Simple Earn / VIP Loan / 现货杠杆）→ 列表 → 持仓 → 预览 → 申购/借币 → 用 `getUserAsset` 核对资金。

### A. 结构化流水线（DAG）

| 步骤 | 动作 |
|------|------|
| **产品线** | 零售赚币 → `simple-earn`；VIP 质押借贷 → `vip-loan`；现货杠杆借还 → `margin-trading`（与 VIP 不同）。 |
| **Simple Earn** | `simple-earn/account` → `flexible/list` 或 `locked/list` → `subscriptionPreview` → 确认 → `subscribe`；赎回前 `position` → `redeem`。 |
| **VIP Loan** | `loanable/data` + `collateral/data` + `interestRate` → 确认 → `borrow`；存续 `ongoing/orders`，还款 `repay` / 续借 `renew`。 |
| **资金核对** | 操作前后 `getUserAsset`（及 `get-funding-asset`）与 `account-and-asset-management.md` 对齐。 |

### B. 接口级速查

**Base**：`https://api.binance.com`（均需签名 USER_DATA，除非注明）。

1. **简单赚币（`simple-earn`）**
   - 总览：`GET /sapi/v1/simple-earn/account`
   - 活期列表：`GET /sapi/v1/simple-earn/flexible/list`；定期列表：`GET /sapi/v1/simple-earn/locked/list`
   - 持仓：`GET /sapi/v1/simple-earn/flexible/position`、`GET /sapi/v1/simple-earn/locked/position`
   - 申购预览：`GET /sapi/v1/simple-earn/flexible/subscriptionPreview`、`GET /sapi/v1/simple-earn/locked/subscriptionPreview`
   - 申购：`POST /sapi/v1/simple-earn/flexible/subscribe`、`POST /sapi/v1/simple-earn/locked/subscribe`
   - 赎回：`POST /sapi/v1/simple-earn/flexible/redeem`、`POST /sapi/v1/simple-earn/locked/redeem`
   - BFUSD/RWUSD 子模块：见 SKILL 中 `/sapi/v1/bfusd/*`、`/sapi/v1/rwusd/*`

2. **VIP 借贷（`vip-loan`）**
   - 利率与可借资产：`GET /sapi/v1/loan/vip/request/interestRate?loanCoin=`、`GET /sapi/v1/loan/vip/loanable/data`
   - 抵押物：`GET /sapi/v1/loan/vip/collateral/data`
   - 借币：`POST /sapi/v1/loan/vip/borrow`（`loanAccountId`、`loanCoin`、`loanAmount`、`collateralAccountId`、`collateralCoin`、`isFlexibleRate` 等）
   - 还款/续借：`POST /sapi/v1/loan/vip/repay`、`POST /sapi/v1/loan/vip/renew`
   - 在借订单：`GET /sapi/v1/loan/vip/ongoing/orders`

3. **资金核对（`assets`）**
   - `POST /sapi/v3/asset/getUserAsset`、`POST /sapi/v1/asset/get-funding-asset`：理财申购前后余额。

4. **与杠杆混淆时（`margin-trading`）**
   - 现货杠杆借还：`POST /sapi/v1/margin/borrow-repay`；与 VIP 借贷产品线不同。

---

## 使用指南

### 结构化要点

- **先分产品再调接口**：勿用 `vip-loan` 响应解释 Simple Earn 申购，避免用户误解利率与风险。
- **申购/借币前后各查一次余额**：便于向用户展示「前后差」与记录是否一致。

### 产品与账户

- **产品分流**：零售赚币走 `simple-earn`；机构式 VIP 质押借贷走 `vip-loan`；现货杠杆是 `margin-trading` 的另一套 `/sapi/v1/margin/*`。
- **与 `account-and-asset-management.md`**：查「活期/定期仓位」以 `simple-earn/position` 为主，通用余额以 `getUserAsset` 为辅。
