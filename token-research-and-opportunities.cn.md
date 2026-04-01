# Token Research and Opportunities（特定资产与投资机会研究）

## Description

**任务说明**：山寨币与板块（AI、DeFi、Meme）、新盘与 Alpha、盈亏比与入场价讨论等「研究向」意图；Skills 提供**数据、信号、审计与排行榜**，Agent 负责逻辑框架与风险提示，不给出确定性投资建议。

**典型线上意图**：山寨币投资建议；新币与趋势；AI 概念币行情；DeFi 资金流入；Meme 与新盘；PRL 等具体标的策略与买入价；盈亏比与重仓判断等。

---

## 推荐 Skills 组合

| 角色 | Skill | 用途 |
|------|--------|------|
| 主路径 | `query-token-info` | 价格、成交、K 线 |
| 主路径 | `crypto-market-rank` | 板块热度与相对强弱 |
| 主路径 | `trading-signal` | 聪明钱与信号侧写 |
| 主路径 | `query-token-audit` | 合约与跑路风险 |
| 场景 | `meme-rush` | Meme 与新盘追踪 |
| 场景 | `alpha` | Binance Alpha 代币相关 |
| 场景 | `binance-tokenized-securities-info` | RWA/美股代币化研究 |

---

## Plan（执行计划）

> 与 `Task_upgrade_advice.cn.md` §6 对齐：**定位 → 排雷（审计）→ 市场位置 → 行为/叙事**；RWA 独立；收口为「事实 + 假设」；下单转 `trading-execution.cn.md`。

### A. 结构化流水线（DAG）

| 步骤 | 动作 |
|------|------|
| **定位** | `search/ai` 或关键词 → `chainId`+`contractAddress`；Alpha 语境并行 `alpha` 的 `token/list` + `ticker`/`klines`。 |
| **排雷** | `query-token-audit`：**不通过**则全文以风险披露为主，少谈「机会」。 |
| **市场位置** | `unified/rank`（关键词/板块 filter）+ `inflow/rank`（`tagType=2`）→ 回答分位、相对强弱。 |
| **行为与叙事** | `trading-signal`；Meme：`meme-rush` 或 exclusive rank；社交热度可选 `social/hype`。 |
| **RWA** | 仅走 `binance-tokenized-securities-info` 列表→动态→K 线，**不**与 Meme 混写。 |
| **收口** | **事实层（接口）+ 假设层（风险与仓位区间）**；执行下单转 `trading-execution.cn.md`。 |

### B. 接口级速查

1. **标的定位**
   - `query-token-info`：`GET .../token/search/ai?keyword=`；确认 `chainId` + `contractAddress`。
   - `alpha`（仅 Alpha 语境）：`GET https://www.binance.com/bapi/defi/v1/public/alpha-trade/get-exchange-info`；`GET https://www.binance.com/bapi/defi/v1/public/alpha-trade/ticker?symbol=`；`GET https://www.binance.com/bapi/defi/v1/public/alpha-trade/klines`；`GET https://www.binance.com/bapi/defi/v1/public/wallet-direct/buw/wallet/cex/alpha/all/token/list`（`symbol` 形如 `ALPHA_175USDT`，来自 Token List）。

2. **行情与板块**
   - `crypto-market-rank`：`POST .../unified/rank/list/ai`（`rankType` 10/11/20/40、`keywords`/`excludes` 筛选）。
   - 聪明钱流入：`POST .../tracker/wallet/token/inflow/rank/query/ai`（`tagType: 2`）。

3. **链上信号与风险**
   - `query-token-audit`：`POST .../security/token/audit`（必传 UUID `requestId`）。
   - `trading-signal`：`POST .../smart-money/ai`。
   - Meme：`POST .../pulse/rank/list/ai`（meme-rush）；或 `crypto-market-rank` 的 `GET .../exclusive/rank/list/ai`。

4. **RWA/美股代币**
   - `binance-tokenized-securities-info`：`GET https://www.binance.com/.../rwa/stock/detail/list/ai` → 动态数据/K 线按 SKILL 工作流（API 1→5/6）。

5. **下单**：转 `trading-execution.cn.md`（`spot`/`fapi`/`dapi`/`eapi` 等），本任务不执行交易 REST。

---

## 使用指南

### 结构化要点

- **接口非指令**：不得把返回字段直接写成「应买入/应卖出」。
- **审计冲突**：`auditInfo` 与专项 `query-token-audit` 冲突时，**以专项审计为准**。

### 接口与字段

- **研究向输出**：引用接口返回字段（如 `riskLevelEnum`、`inflow`、`percentChange24h`），避免编造未返回的指标。
- **Alpha**：使用 `bapi/defi/v1/public/alpha-trade/*` 与 CEX Alpha 列表，勿与 Web3 `rankType=20` 混淆（后者为榜单「Alpha 代币」维度）。
