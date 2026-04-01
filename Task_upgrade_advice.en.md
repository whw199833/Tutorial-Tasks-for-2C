# Task upgrade advice — default execution DAG (overview)

This document summarizes the **default structured pipelines** referenced by each Task’s **Plan §A**. Per-Task API paths and parameters are in each Task’s **§B** and in `binance-skills-hub` `SKILL.md` files.

| § | Task / theme | Default pipeline (summary) |
|---|----------------|----------------------------|
| **1** | [account-and-asset-management.en.md](./account-and-asset-management.en.md) | Scope → balance snapshot → reconcile deposits/withdrawals/transfers → sub-account / fiat branches → concise output. |
| **2** | [api-authorization-and-debugging.en.md](./api-authorization-and-debugging.en.md) | Classify errors → verify permissions → read path per market → sub-account / IP → signing & environment → supply-chain tools (`skill-vetter` / `healthcheck`). |
| **3** | [trading-execution.en.md](./trading-execution.en.md) | Lock **one** market → read-only orders/positions → precision & rules before writes → track algo sub-orders; quotes from [market-data-and-analysis.en.md](./market-data-and-analysis.en.md). |
| **4** | [market-data-and-analysis.en.md](./market-data-and-analysis.en.md) | Broad first, then narrow: ranks → Top N → `meta` + `dynamic` → 1–3 symbols multi-interval K-lines → optional inflow / smart-money; RWA separate. Scheduled pulls: **§B.C** (Python / Shell). |
| **5** | [onchain-signals-and-security.en.md](./onchain-signals-and-security.en.md) | **Audit first** if contract exists → signals → liquidity check → address pagination; Meme narrative alignment optional. Scheduled monitoring/alerts: default scripts in **§B.C** (Python / Shell). |
| **6** | [token-research-and-opportunities.en.md](./token-research-and-opportunities.en.md) | Locate symbol → audit (risk) → market position → behavior / narrative → RWA isolated; execution → [trading-execution.en.md](./trading-execution.en.md). |
| **7** | [product-help-and-order-lookup.en.md](./product-help-and-order-lookup.en.md) | FAQ first → APIs only for “my orders / grid”; confirm spot / futures / algo before read-only order calls. |
| **8** | [token-deployment-and-onchain-pay.en.md](./token-deployment-and-onchain-pay.en.md) | No one-click deploy REST; on-chain pay: pairs → payment → quote → address/network → pre-order → poll; post-deploy: search + audit + address positions. |
| **9** | [campaigns-and-rewards.en.md](./campaigns-and-rewards.en.md) | Official rules first → optional token quotes → optional user self-check volume; discussion = no API. |
| **10** | [binance-square-posting.en.md](./binance-square-posting.en.md) | Draft & compliance → optional market facts → single `content/add`; scheduled posts default to **§B.C** (Python / Shell + cron). |
| **11** | [education-and-learning.en.md](./education-and-learning.en.md) | Concepts first → on request: search → dynamic → K-line chain; optional `skill-creator` for reusable tutorials. |
| **12** | [simple-earn-and-vip-loan.en.md](./simple-earn-and-vip-loan.en.md) | Pick product line (Simple Earn / VIP Loan / margin) → lists → positions → preview → subscribe/borrow → `getUserAsset` to verify. |

Cross-links: asset depth → [account-and-asset-management.en.md](./account-and-asset-management.en.md); trades → [trading-execution.en.md](./trading-execution.en.md); market context → [market-data-and-analysis.en.md](./market-data-and-analysis.en.md).
