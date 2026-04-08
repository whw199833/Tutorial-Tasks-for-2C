# Account and Asset Management（账户与资产管理）

## Description

**任务说明**：帮助用户完成主/子账户资产查询、余额与分布核对、充值/提现/划转状态追踪，以及子账户管理能力相关的咨询与操作辅助。

**典型线上意图**（摘自 intent 分类）：查询币安账户资产与交易状态；主账户余额确认；授权代操作及资金状况核查；账户余额与资产分布；资金去向追踪；开户与交易操作咨询；账号功能关闭与退款相关咨询（信息侧）。

**Skills 边界**：资产与划转以 `assets` 为主；子账户以 `sub-account` 为主；法币流水以 `fiat` 为辅；若用户同时问合约/统一账户保证金，可衔接 `trading-execution.cn.md` 中的衍生品 Skills。

---

## 推荐 Skills 组合

| 角色 | Skill | 用途 |
|------|--------|------|
| 主路径 | `assets` | 余额、充值提现、划转、资产分布 |
| 主路径 | `sub-account` | 子账户列表、权限与划转关系 |
| 辅助 | `fiat` | 法币充提记录核对 |
| 衔接 | `derivatives-trading-portfolio-margin` / `derivatives-trading-portfolio-margin-pro` | 仅当用户明确问统一账户/组合保证金视图时 |

---

## Plan（执行计划）

> 与 `Task_upgrade_advice.cn.md` §1 对齐：先定界 → 快照 → 流水对齐 → 子账户/法币分支 → 收口输出。

### 状态确认与不支持时的沟通

- **规划前需确认**：定界范围（主账户/资金/子账户/法币）；与问题相关的资产与**时间窗**是否清楚。
- **若流水与快照对不上、或接口无权限**：① 说明已核对项与缺口；② 追问用户：是否还有其它钱包/子账户、是否可提供某笔 txId 或 App 内订单时间；③ 不编造「资金去向」；政策/关户类引导官网与客服。
- **跨 Task 原则**：全局与「不支持时询问用户」流程见 [Task_upgrade_advice.cn.md](./Task_upgrade_advice.cn.md) 开篇。

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
| **定界** | 区分：仅主账户现货 / 含 Funding / 含子账户 / 含法币订单 / 含合约或 PM（后者转 `trading-execution.cn.md`）。 |
| **余额快照** | `getUserAsset` 或 `wallet/balance` → 答「某币剩多少」可结束；问「钱从哪来/到哪去」进入流水。 |
| **流水对齐** | 在时间窗内：`deposit/hisrec`、`withdraw/history`、`asset/transfer`（GET）与快照**交叉核对**（大额变动应对应记录）。 |
| **子账户** | 仅当明确子账户 email 或主子划转：`sub-account/list` → `assets` → 必要时 `universalTransfer` 历史。 |
| **法币** | 渠道/限额：Public `get-capabilities` / `get-price`；**我的订单**：SAPI `fiat/orders`、`fiat/payments` 与资产时间线对齐。 |
| **收口** | 按钱包类型分块：**余额 + 近期 3～5 条关键流水**；退款/关户等政策只陈述数据事实并引用官网。 |

### B. 接口级速查

以下 HTTP 路径与参数以 Skill `assets`、`sub-account`、`fiat` 的 SKILL.md（`binance-skills-hub/skills/binance/<skill>/`）为准；Base：`https://api.binance.com`（REST 均需 `X-MBX-APIKEY` + 签名参数 `timestamp`/`signature`，除非标明 Public）。

**补充（现货快照）**：若环境启用 **`binance-cli`**，可用 `binance` Skill（`skills/binance/binance/`）的 `binance-cli spot get-account` 与 **REST** `assets` / `spot` 的账户视图**对照**；充提划转仍以 `assets` 为主。

1. **主账户现货/资金余额**
   - `POST /sapi/v3/asset/getUserAsset`：用户各资产余额（可按 `asset` 过滤）。
   - `GET /sapi/v1/asset/wallet/balance`：按 `quoteAsset` 汇总钱包视图。
   - `GET /api/v3/account`（`spot` Skill）：现货账户余额与挂单占用（若需与现货订单联动）。

2. **充值 / 提现 / 划转追踪**
   - `GET /sapi/v1/capital/deposit/hisrec`：充值历史（`coin`、`status`、`startTime`/`endTime` 等）。
   - `GET /sapi/v1/capital/withdraw/history`：提现历史。
   - `GET /sapi/v1/asset/transfer`（GET）：Universal Transfer 历史；`POST /sapi/v1/asset/transfer`：划转（`type`、`asset`、`amount`，及 `fromSymbol`/`toSymbol` 等按文档）。

3. **资金钱包 / 快照**
   - `POST /sapi/v1/asset/get-funding-asset`：Funding Wallet 资产。
   - `GET /sapi/v1/accountSnapshot`：日快照（`type`：SPOT/MARGIN 等）。

4. **子账户（主账户视角）**
   - `GET /sapi/v1/sub-account/list`：子账户列表（`email`、`page`、`limit`）。
   - `GET /sapi/v4/sub-account/assets` 或 `GET /sapi/v3/sub-account/assets`：指定子账户 `email` 的资产。
   - `GET /sapi/v1/sub-account/universalTransfer`（GET）：主子划转历史；`POST .../universalTransfer`：Universal Transfer（`fromAccountType`/`toAccountType`：`SPOT`、`USDT_FUTURE`、`COIN_FUTURE`、`MARGIN`、`ISOLATED_MARGIN` 等）。

5. **法币订单与支付记录（需 API Key）**
   - `GET /sapi/v1/fiat/orders`：`transactionType` 等，法币充提订单历史。
   - `GET /sapi/v1/fiat/payments`：法币支付流水。
   - **公开能力（无 Key）**：`https://www.binance.com/bapi/fiat/v1/public/fiatpayment/agent/get-capabilities?country={CC}`，以及同前缀的 `get-buy-and-sell-payment-methods`、`get-deposit-and-withdraw-payment-methods`、`get-price`（仅能力/渠道/参考价，非账户流水）。

---

## 使用指南

### 结构化要点

- **禁止跳步**：未搞清「主 vs 子」前不要并行打满全部 SAPI；子账户仅在对话明确后出现。
- **流水优先于猜测**：用户追问资金去向时，必须用充值/提现/划转之一对齐时间线，再解释。

### 接口与鉴权

- **鉴权**：`assets` / `sub-account` / SAPI 法币端点使用 `BINANCE_API_KEY`、`BINANCE_SECRET_KEY`；签名算法见各 Skill `references/authentication.md`；`User-Agent` 按 SKILL 要求（如 `binance-wallet/1.1.0 (Skill)`）。
- **先 REST 资产再子账户**：仅主账户余额用 `getUserAsset` 即可；涉及子账户邮箱或划转时再调用 `sub-account` 的 `list` / `assets` / `universalTransfer`。
- **法币公开 API 与 SAPI 分工**：询价/渠道用 `bapi/fiat/.../agent/*`；**用户账户级**法币订单用 `GET /sapi/v1/fiat/orders`、`/sapi/v1/fiat/payments`。
- **主网资金操作**：Skill 要求主网交易前用户输入 `CONFIRM`；代操作类须合规提示。
- **与 `trading-execution.cn.md`**：下单后查单不在本 Task；现货订单走 `spot` 的 `/api/v3/order`、`/api/v3/openOrders`。
