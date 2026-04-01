# Education and Learning（教育与学习）

## Description

**任务说明**：学习计划、资料清单、术语定义、投资知识、语言学习等；以**讲解与结构化输出**为主，可选用行情/数据 Skills 作为教学案例，不强制调用。

**典型线上意图**：股票/投研学习计划与资料；交易术语定义；语言学习与表达分析。

---

## 推荐 Skills 组合

| 场景 | Skill | 用途 |
|------|--------|------|
| 案例演示 | `query-token-info` | 用真实行情解释 K 线、成交量等概念 |
| 案例演示 | `crypto-market-rank` | 解释排行榜、热度与波动 |
| 扩展 | `skill-creator` | 用户希望把知识固化成可复用 Agent Skill 时 |
| 一般 | — | 术语与通识无需 Skill |

---

## Plan（执行计划）

> 与 `Task_upgrade_advice.md` §11 对齐：**概念先行** → 用户要实例时再走 **search → dynamic → K 线** 演示链；榜单教学单次 `unified/rank`；固化教程用 `skill-creator`。

### A. 结构化流水线（DAG）

| 步骤 | 动作 |
|------|------|
| **默认** | 术语、学习计划、资料结构 → **无 API**。 |
| **演示链** | 选一主流币：`search` → `dynamic` 解释字段 → **再** `K-line` 解读 OHLCV 数组。 |
| **榜单教学** | 一次 `unified/rank`，解释 `rankType` / `period` / `sortBy`。 |
| **边界** | 声明**教学不构成买卖建议**；需要可复用教程时再 `skill-creator`。 |

### B. 接口级速查

1. **术语与通识**：零接口；直接讲解。

2. **可选案例：单币行情**
   - `GET https://web3.binance.com/bapi/defi/v5/public/wallet-direct/buw/wallet/market/token/search/ai?keyword=BTC`
   - `GET .../bapi/defi/v4/public/wallet-direct/buw/wallet/market/token/dynamic/info/ai?chainId=&contractAddress=`（解释成交量、涨跌、持有人等字段）。

3. **可选案例：榜单**
   - `POST .../unified/rank/list/ai`，展示 `rankType`、`period` 对排序的影响。

4. **可选案例：K 线**
   - `GET https://dquery.sintral.io/u-kline/v1/k-line/candles`（参数见 `query-token-info` SKILL：OHLCV 数组下标含义）。

5. **固化教程为 Skill**：使用 Cursor `skill-creator` Skill（非 Binance HTTP）。

---

## 使用指南

### 结构化要点

- **按需才开接口**：用户未要求实例则保持零请求，减少认知负荷。
- **演示顺序固定**：先解释字段含义，再打开 K 线，避免跳过「动态数据」直接看图。

### 合规表述

- **中性**：接口返回值不得解释为「必涨/必买」。
