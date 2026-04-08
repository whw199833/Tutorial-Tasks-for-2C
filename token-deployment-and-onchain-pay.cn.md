# Token Deployment and On-chain Pay（代币发行与部署 · 链上）

## Description

**任务说明**：在 BNB 链等创建、发射、部署代币及社交链接集成等意图。当前 Binance Skills 以**交易、资产、行情、审计**为主，**不包含完整合约 IDE 或一键发币部署**；可用 Skills 做支付、代币信息与安全侧核查。

**典型线上意图**：BNB 链创建和发射代币；代部署与命名；集成社交媒体链接。

---

## 推荐 Skills 组合

| 角色 | Skill | 用途 |
|------|--------|------|
| 支付/买币 | `onchain-pay-open-api` | 法币买币或向链上地址发送（若流程适用） |
| 信息 | `query-token-info` | 已有代币信息参考 |
| 安全 | `query-token-audit` | 部署前后合约风险意识教育 |
| 地址 | `query-address-info` | 部署相关地址持仓查看 |

**binance-skills-hub**：`onchain-pay-open-api` → **`skills/binance/onchain-pay/SKILL.md`**（`name` 为 `onchain-pay-open-api`）；其余 Web3 Skills → **`skills/binance-web3/`**。

---

## Plan（执行计划）

> 与 `Task_upgrade_advice.cn.md` §8 对齐：部署无一键 REST；支付侧 **pairs → payment → quote → address/network → pre-order → poll**；部署后 **search + audit + 地址持仓** 做事后验证。

### 状态确认与不支持时的沟通

- **规划前需确认**：支付路径下的 **法币/币对、金额、收款地址与网络**；`estimated-quote` 前是否具备足够资金（可衔接 `assets`）。
- **若链上支付失败或限额不足**：① 说明错误与限额；② 追问：是否换支付方式/网络、是否调整金额；③ 在将 `pre-order`/`order` 推进至终态前，须由用户确认地址与网络无误。
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
| **需求** | 法币买币打到地址 vs 已有币发送 → 决定 `pre-order` 的 `customization`（如 `SEND_PRIMARY`）。 |
| **能力** | `trading-pairs` → `payment-method-list` → `estimated-quote`（同一 fiat/crypto/amount 口径）。 |
| **下单前** | 必填或配置 **`address` + `network`**；`crypto-network` 看费用与限额。 |
| **下单** | `pre-order` → `order` 轮询至终态。 |
| **部署后** | 有合约则 `search` + `audit` + `query-address-info`（部署者钱包），**不**替代 Remix/Hardhat。 |

### B. 接口级速查

1. **Onchain Pay（`onchain-pay-open-api`，需 RSA 签名，见 Skill `scripts/sign_and_call.sh`）**
   - `papi/v1/ramp/connect/buy/trading-pairs`：支持法币/币对。
   - `papi/v1/ramp/connect/buy/payment-method-list`（v2：`papi/v2/ramp/connect/buy/payment-method-list`）：支付方式与限额。
   - `papi/v1/ramp/connect/buy/estimated-quote`：`fiatCurrency`、`requestedAmount`、`payMethodCode`、`amountType` 等。
   - `papi/v1/ramp/connect/gray/buy/pre-order`：`externalOrderId`、`merchantCode`、`merchantName`、`ts`，及 `address`、`network`、`customization`（如 `SEND_PRIMARY`、`ON_CHAIN_PROXY_MODE`）。
   - `papi/v1/ramp/connect/order`：按 `externalOrderId` 查单。
   - `papi/v1/ramp/connect/crypto-network`：链、手续费与限额。

2. **部署后代币信息（`query-token-info`）**
   - `GET https://web3.binance.com/bapi/defi/v5/public/wallet-direct/buw/wallet/market/token/search/ai?keyword=`（合约地址）。

3. **安全（`query-token-audit`）**
   - `POST https://web3.binance.com/bapi/defi/v1/public/wallet-direct/security/token/audit`（`binanceChainId`、`contractAddress`、`requestId`）。

4. **部署者地址持仓（`query-address-info`）**
   - `GET https://web3.binance.com/bapi/defi/v3/public/wallet-direct/buw/wallet/address/pnl/active-position-list/ai?address=&chainId=&offset=`。

5. **合约编写/部署**：无专用 Binance REST；Remix/Hardhat 等链下工具不由本 Skill 集覆盖。

---

## 使用指南

### 结构化要点

- **链上部署与 Onchain Pay 分离**：合约编译/部署不在本 API 流程内；Pay 只解决资金进出链上地址。

### 接口与凭证

- **Onchain Pay**：配置 `BASE_URL`、`CLIENT_ID`、`API_KEY`、RSA PEM；`pre-order` 通常要求 `address` + `network`（或 `.local.md` 默认）。
- **与 `onchain-signals-and-security.cn.md`**：新代币上线后持续用 `query-token-audit` 与 `query-token-info`。
