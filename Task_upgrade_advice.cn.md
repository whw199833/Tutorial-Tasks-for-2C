# Task 升级建议 — 默认执行 DAG（总览）

本文汇总各 Task **Plan §A** 中引用的**默认结构化流水线**。具体 HTTP 路径与参数以各 Task **§B** 及 `binance-skills-hub` 中对应 `SKILL.md` 为准。

---

## 规划前：先确认账户状态（跨 Task）

在对用户任务 **plan**（制定执行计划或给出交易/产品侧建议）之前，Agent **应先**确认用户当前账户状态，再判断能否支持其操作或给出可执行建议：

- **余额与可用资金**：对应市场/账户类型下的可用余额、保证金占用（合约/杠杆/统一账户等）；避免在资金不足或币种错配时推进下单类计划。
- **挂单与其他在途单**：未成交挂单、算法单、网格、条件单等；避免与已有仓位或策略冲突、重复下单。
- **其他相关状态**：理财/借贷占用、冻结、子账户与划转前提等；需要时衔接 [account-and-asset-management.cn.md](./account-and-asset-management.cn.md) 与各 Task §A。

若暂时无法通过工具拉取上述信息，应 **向用户说明缺失项** 或引导其自查后再给出计划/建议；**不应**仅凭行情或假设余额直接给出可执行交易路径。

### 状态不支持或信息不足时：询问用户的建议流程

1. **说明**：简要说明已掌握的事实与缺口（例如：余额不足、未确认市场类型、接口无权限、与活动统计口径可能不一致）。
2. **追问**：向用户提出可选项或确认项（是否已充值/划转、是否可撤销某笔挂单、是否提供订单号/交易对/时间范围、是否在 App 内核对活动进度或 API 权限等）。
3. **收口**：在用户补充或完成自查前，**不推进**不可逆操作建议（下单、划转、申购、借币、链上支付等）；可给出仅基于公开信息或假设的**条件化**表述（「若余额为 X、且无冲突挂单，则可…」）。

各 Task 文档 **Plan** 内另有与本意图相关的细化；下表为默认流水线摘要。

---

| § | Task / 主题 | 默认流水线（摘要） |
|---|----------------|---------------------|
| **1** | [account-and-asset-management.cn.md](./account-and-asset-management.cn.md) | 定界 → 余额快照 → 充提划转对齐 → 子账户/法币分支 → 收口输出。 |
| **2** | [api-authorization-and-debugging.cn.md](./api-authorization-and-debugging.cn.md) | 归类错误 → 权限真值 → 分市场读通路 → 子账户/IP → 签名环境 → 供应链工具（`skill-vetter` / `healthcheck`）。 |
| **3** | [trading-execution.cn.md](./trading-execution.cn.md) | **唯一**市场锁定 → 只读查单/持仓 → 写单前精度与规则 → 算法子单追踪；行情来自 [market-data-and-analysis.cn.md](./market-data-and-analysis.cn.md)。现货/闪兑/U 本位可对照 **`binance`**（`binance-cli`，见该 Task §C）。 |
| **4** | [market-data-and-analysis.cn.md](./market-data-and-analysis.cn.md) | 先广后窄：榜单 → Top N → `meta` + `dynamic` → 1～3 标的 K 线 → 可选流入/聪明钱；RWA 独立。定时拉行情见该文件 **§B.C**（Python / Shell）。 |
| **5** | [onchain-signals-and-security.cn.md](./onchain-signals-and-security.cn.md) | **有合约则先审计** → 信号 → 流动性检查 → 地址分页；Meme 叙事可对齐。定时监控/告警默认脚本见 **§B.C**（Python / Shell）。 |
| **6** | [token-research-and-opportunities.cn.md](./token-research-and-opportunities.cn.md) | 定位 → 排雷（审计）→ 市场位置 → 行为/叙事 → RWA 独立；下单转 [trading-execution.cn.md](./trading-execution.cn.md)。 |
| **7** | [product-help-and-order-lookup.cn.md](./product-help-and-order-lookup.cn.md) | 先 FAQ 后接口；仅「查我的单/网格」时走读单；先确认现货/合约/算法。 |
| **8** | [token-deployment-and-onchain-pay.cn.md](./token-deployment-and-onchain-pay.cn.md) | 无一键部署 REST；支付：pairs → payment → quote → address/network → pre-order → poll；部署后 search + audit + 地址持仓。 |
| **9** | [campaigns-and-rewards.cn.md](./campaigns-and-rewards.cn.md) | 规则以官网为准 → 可选代币行情 → 可选用户自查交易量；讨论类无 API。 |
| **10** | [binance-square-posting.cn.md](./binance-square-posting.cn.md) | 撰稿定稿 → 可选行情事实 → 单次 `content/add`；定时发帖默认 **§B.C**（Python / Shell + cron）。 |
| **11** | [education-and-learning.cn.md](./education-and-learning.cn.md) | 概念先行 → 用户要实例再走 search → dynamic → K 线；可复用教程用 `skill-creator`。 |
| **12** | [simple-earn-and-vip-loan.cn.md](./simple-earn-and-vip-loan.cn.md) | 先定产品线（Simple Earn / VIP Loan / 杠杆）→ 列表 → 持仓 → 预览 → 申购/借币 → `getUserAsset` 核对。 |

交叉引用：资产深度 → [account-and-asset-management.cn.md](./account-and-asset-management.cn.md)；交易 → [trading-execution.cn.md](./trading-execution.cn.md)；行情 → [market-data-and-analysis.cn.md](./market-data-and-analysis.cn.md)。
