# skills-for-2C

面向 **Binance AI Pro（Clawbot）** 类场景：将线上用户意图抽象为若干 **Task**，并为每类 Task 说明 **适用 Skills**、**结构化执行顺序（DAG）** 与 **REST/接口速查**，便于 Agent 按步骤选 Skill、少平铺调用。

---

## 仓库里有什么

| 内容 | 说明 |
|------|------|
| **Task 文档**（`*.md`，英文文件名） | 每类意图一篇：`Description`、推荐 Skills 组合、**Plan（§A DAG + §B 接口级速查）**、`使用指南`。 |
| **[Task_upgrade_advice.md](./Task_upgrade_advice.md)** | 跨 Task 的 **默认执行 DAG** 总览（与各文件 §A 对应）；接口细节仍以各 Task §B 与 `binance-skills-hub` 为准。 |
| **意图分类参考** | 线上意图大类与 query 示例见：`intent_category.json`。 |

---

## 文档怎么读（推荐顺序）

1. **定意图**：在 `intent_category.json` 里找到大类，再在下表映射到 **Task 文件名**。
2. **读 DAG**：打开对应 Task，先看 **Plan → §A 结构化流水线**，理清「先做什么、再收窄什么」。
3. **查接口**：同一 Task 内 **§B 接口级速查** 与 `binance-skills-hub` 中对应 `SKILL.md` 一致，用于具体 path/参数。

---

## Task 文档索引（意图 → 英文文件名 → 核心 Skills）

| Task 文件 | 意图大类（intent_category） | 核心 Skills |
|-----------|------------------------------|-------------|
| [account-and-asset-management.md](./account-and-asset-management.md) | 账户与资产管理 | assets, sub-account, fiat |
| [api-authorization-and-debugging.md](./api-authorization-and-debugging.md) | API 与系统授权配置 | assets, derivatives-*, sub-account, healthcheck, skill-vetter |
| [trading-execution.md](./trading-execution.md) | 交易策略与执行 | spot, convert, algo, p2p, margin-trading, derivatives-* |
| [market-data-and-analysis.md](./market-data-and-analysis.md) | 市场分析与行情查询 | query-token-info, crypto-market-rank, trading-signal, meme-rush, binance-tokenized-securities-info |
| [onchain-signals-and-security.md](./onchain-signals-and-security.md) | 链上数据与智能资金监控 | trading-signal, query-token-audit, query-address-info, query-token-info, meme-rush |
| [token-research-and-opportunities.md](./token-research-and-opportunities.md) | 特定资产与投资机会研究 | query-token-info, crypto-market-rank, trading-signal, query-token-audit, meme-rush, alpha |
| [product-help-and-order-lookup.md](./product-help-and-order-lookup.md) | 工具与功能使用咨询 | spot / algo / derivatives-*（查单）, skill-creator, skill-vetter |
| [token-deployment-and-onchain-pay.md](./token-deployment-and-onchain-pay.md) | 代币发行与部署 | onchain-pay-open-api, query-token-info, query-token-audit, query-address-info |
| [campaigns-and-rewards.md](./campaigns-and-rewards.md) | 活动与激励 | alpha（场景补充） |
| [binance-square-posting.md](./binance-square-posting.md) | 内容创作与社交媒体 | square-post |
| [education-and-learning.md](./education-and-learning.md) | 教育与学习 | query-token-info（案例）, skill-creator（可选） |
| [simple-earn-and-vip-loan.md](./simple-earn-and-vip-loan.md) | 理财与借贷（补充） | simple-earn, vip-loan, assets |

**说明**：

- `weather` 为独立工具类 Skill，可按需单独调用，未单独成篇。
- 与「系统环境安全」相关时，可结合 [api-authorization-and-debugging.md](./api-authorization-and-debugging.md) 中的 `healthcheck`。
- 链上/监控类任务中 **有合约地址时优先审计**（`query-token-audit`），见 [onchain-signals-and-security.md](./onchain-signals-and-security.md) §A。

---

## Skill 类型与名称（与 binance-skills-hub 对齐）

共 **28** 个 Skill（名称以 hub 目录为准），按类别汇总如下。

**现货与交易**：`spot`，`convert`，`algo`，`p2p`

**衍生品**：`derivatives-trading-usds-futures`，`derivatives-trading-coin-futures`，`derivatives-trading-options`，`derivatives-trading-portfolio-margin`，`derivatives-trading-portfolio-margin-pro`，`margin-trading`

**资产与理财**：`assets`，`simple-earn`，`vip-loan`，`fiat`

**行情与数据**：`query-token-info`，`crypto-market-rank`，`binance-tokenized-securities-info`，`trading-signal`，`meme-rush`

**安全与审计**：`query-token-audit`，`query-address-info`，`skill-vetter`，`healthcheck`

**社交与支付**：`square-post`，`onchain-pay-open-api`，`alpha`

**账户与工具**：`sub-account`，`skill-creator`，`weather`

---

## 合规与边界

- Task 文档仅作 **能力与路由** 说明，不构成投资建议；涉及交易、借贷、活动规则时，以 **Binance 官方页面与产品规则** 为准。
- 活动与激励类（[campaigns-and-rewards.md](./campaigns-and-rewards.md)）**无**独立活动全文 API，规则以官网/App 为准。

---

## 变更与维护

- 新增或调整意图时：在 `intent_category.json` 侧更新后，评估是否新增 Task 或改映射表。
