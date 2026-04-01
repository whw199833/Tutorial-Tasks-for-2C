# skills-for-2C

For **Binance AI Pro (Clawbot)**–style scenarios: map live user intents to **Tasks**, and for each Task document **recommended Skills**, a **structured execution order (DAG)**, and **REST / API quick reference**, so agents can pick Skills step-by-step with fewer flat tool calls.

---

## What is in this repo

| Item | Description |
|------|-------------|
| **Task docs** (`*.en.md` / `*.cn.md`; English base filenames) | One doc per intent: `Description`, recommended Skill combos, **Plan (§A DAG + §B API quick reference)**, and **Usage**. |
| **[Task_upgrade_advice.en.md](./Task_upgrade_advice.en.md)** | Cross-Task **default execution DAG** overview (aligned with §A in each file); API details remain in each Task’s §B and in `binance-skills-hub`. |
| **Intent reference** | High-level intent categories and example queries: `intent_category.json`. |

---

## How to read the docs (recommended order)

1. **Identify intent**: Find the category in `intent_category.json`, then map it to a **Task filename** using the table below.
2. **Read the DAG**: Open the Task and read **Plan → §A structured pipeline** first to see “what comes first, what narrows next.”
3. **Look up APIs**: **§B API quick reference** in the same Task matches `SKILL.md` in `binance-skills-hub` for concrete paths and parameters.

---

## Task index (intent → filename → core Skills)

| Task file | Intent category (intent_category) | Core Skills |
|-----------|-----------------------------------|-------------|
| [account-and-asset-management.en.md](./account-and-asset-management.en.md) | Account & asset management | assets, sub-account, fiat |
| [api-authorization-and-debugging.en.md](./api-authorization-and-debugging.en.md) | API & system authorization | assets, derivatives-*, sub-account, healthcheck, skill-vetter |
| [trading-execution.en.md](./trading-execution.en.md) | Trading strategy & execution | spot, convert, algo, p2p, margin-trading, derivatives-* |
| [market-data-and-analysis.en.md](./market-data-and-analysis.en.md) | Market analysis & quotes | query-token-info, crypto-market-rank, trading-signal, meme-rush, binance-tokenized-securities-info |
| [onchain-signals-and-security.en.md](./onchain-signals-and-security.en.md) | On-chain data & smart-money monitoring | trading-signal, query-token-audit, query-address-info, query-token-info, meme-rush |
| [token-research-and-opportunities.en.md](./token-research-and-opportunities.en.md) | Token & opportunity research | query-token-info, crypto-market-rank, trading-signal, query-token-audit, meme-rush, alpha |
| [product-help-and-order-lookup.en.md](./product-help-and-order-lookup.en.md) | Product help & order lookup | spot / algo / derivatives-* (orders), skill-creator, skill-vetter |
| [token-deployment-and-onchain-pay.en.md](./token-deployment-and-onchain-pay.en.md) | Token launch & on-chain pay | onchain-pay-open-api, query-token-info, query-token-audit, query-address-info |
| [campaigns-and-rewards.en.md](./campaigns-and-rewards.en.md) | Campaigns & rewards | alpha (context) |
| [binance-square-posting.en.md](./binance-square-posting.en.md) | Content & social posting | square-post |
| [education-and-learning.en.md](./education-and-learning.en.md) | Education & learning | query-token-info (examples), skill-creator (optional) |
| [simple-earn-and-vip-loan.en.md](./simple-earn-and-vip-loan.en.md) | Earn & lending (supplement) | simple-earn, vip-loan, assets |

**Notes**:

- `weather` is a standalone utility Skill; call it as needed; it is not a separate Task doc.
- For “system / environment security,” combine with `healthcheck` in [api-authorization-and-debugging.en.md](./api-authorization-and-debugging.en.md).
- For on-chain / monitoring tasks, **prefer audit when a contract address exists** (`query-token-audit`); see §A in [onchain-signals-and-security.en.md](./onchain-signals-and-security.en.md).

---

## Skill types and names (aligned with binance-skills-hub)

**28** Skills (names as in the hub), grouped:

**Spot & trading**: `spot`, `convert`, `algo`, `p2p`

**Derivatives**: `derivatives-trading-usds-futures`, `derivatives-trading-coin-futures`, `derivatives-trading-options`, `derivatives-trading-portfolio-margin`, `derivatives-trading-portfolio-margin-pro`, `margin-trading`

**Assets & earn**: `assets`, `simple-earn`, `vip-loan`, `fiat`

**Market data**: `query-token-info`, `crypto-market-rank`, `binance-tokenized-securities-info`, `trading-signal`, `meme-rush`

**Security & audit**: `query-token-audit`, `query-address-info`, `skill-vetter`, `healthcheck`

**Social & pay**: `square-post`, `onchain-pay-open-api`, `alpha`

**Account & tools**: `sub-account`, `skill-creator`, `weather`

---

## Compliance & boundaries

- Task docs describe **capabilities and routing** only; they are not investment advice. For trading, lending, and campaign rules, follow **Binance official pages and product terms**.
- Campaigns & rewards ([campaigns-and-rewards.en.md](./campaigns-and-rewards.en.md)) have **no** full campaign-text API in this skill set; rules are on the website / app.

---

## Changes & maintenance

- When intents change: update `intent_category.json` and decide whether to add Tasks or adjust the mapping table.
