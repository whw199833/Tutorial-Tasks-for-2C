# Task 升级建议 — 默认执行 DAG（总览）

本文汇总各 Task **Plan §A** 中引用的**默认结构化流水线**。具体 HTTP 路径与参数以各 Task **§B** 及 `binance-skills-hub` 中对应 `SKILL.md` 为准。

| § | Task / 主题 | 默认流水线（摘要） |
|---|----------------|---------------------|
| **1** | [account-and-asset-management.cn.md](./account-and-asset-management.cn.md) | 定界 → 余额快照 → 充提划转对齐 → 子账户/法币分支 → 收口输出。 |
| **2** | [api-authorization-and-debugging.cn.md](./api-authorization-and-debugging.cn.md) | 归类错误 → 权限真值 → 分市场读通路 → 子账户/IP → 签名环境 → 供应链工具（`skill-vetter` / `healthcheck`）。 |
| **3** | [trading-execution.cn.md](./trading-execution.cn.md) | **唯一**市场锁定 → 只读查单/持仓 → 写单前精度与规则 → 算法子单追踪；行情来自 [market-data-and-analysis.cn.md](./market-data-and-analysis.cn.md)。 |
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
