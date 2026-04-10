# Tutorial-Tasks-for-Skills

For **Binance AI Pro (Clawbot)**-style scenarios: map online user intents to **Tasks**, and for each Task list **applicable Skills**, a **structured execution order (DAG)**, and **REST/endpoint quick reference**, so agents pick skills step-by-step instead of flat API calls.

This folder lives under **`skills/Tutorial-Tasks-for-Skills/`** in the **binance-skills-hub** repository. Executable hub skills are **siblings**: **`../binance/`** (REST + `binance-cli` bundle) and **`../binance-web3/`** (market, audit, signals). From the **repository root**, those are `skills/binance/` and `skills/binance-web3/`.

---

## What’s in this folder

| Item | Description |
|------|-------------|
| **Task docs** (`*.md`) | One file per intent: `Description`, recommended Skills, **Plan (§A DAG + §B endpoint quick ref [+ §C `binance-cli` where applicable])**, usage notes. |
| **DAG overview** | **[Task_upgrade_advice.md](./Task_upgrade_advice.md)**: cross-task default pipelines aligned with §A in each Task; HTTP detail stays in each Task §B and the matching hub `SKILL.md`. |
| **Intent taxonomy** | **`intent_category.json`**: intent categories and example queries. |

---

## How to read (recommended)

1. **Pick intent**: Use **`intent_category.json`**, then the **Task index** table below.
2. **Pre-planning account check** (funds, trades, transfers, earn subscribe, borrow): Before **planning**, confirm balance, open/in-flight orders, locks—see **Pre-planning** in [Task_upgrade_advice.md](./Task_upgrade_advice.md). Pure market data, education, or compliance explainers may skip.
3. **Read the DAG**: Open the Task → **Plan → §A** (narrowing order).
4. **Look up endpoints**: Task **§B** ↔ hub **`SKILL.md`**. For **spot / convert / USDS-M** with `binance-cli`, also open **`../binance/binance/SKILL.md`** and `../binance/binance/references/*.md` (repo root: `skills/binance/binance/`).

---

## Task index (intent → file → core skills)

| Task doc | Intent (summary) | Core skills |
|----------|------------------|-------------|
| [account-and-asset-management.md](./account-and-asset-management.md) | Account & assets | assets, sub-account, fiat |
| [api-authorization-and-debugging.md](./api-authorization-and-debugging.md) | API & system auth | assets, derivatives-*, sub-account, healthcheck, skill-vetter |
| [trading-execution.md](./trading-execution.md) | Trading strategy & execution | **binance** (CLI), spot, convert, algo, p2p, margin-trading, derivatives-* |
| [market-data-and-analysis.md](./market-data-and-analysis.md) | Market analysis & quotes | query-token-info, crypto-market-rank, trading-signal, meme-rush, binance-tokenized-securities-info |
| [onchain-signals-and-security.md](./onchain-signals-and-security.md) | On-chain data & smart money | trading-signal, query-token-audit, query-address-info, query-token-info, meme-rush |
| [token-research-and-opportunities.md](./token-research-and-opportunities.md) | Token research & opportunities | query-token-info, crypto-market-rank, trading-signal, query-token-audit, meme-rush, alpha |
| [product-help-and-order-lookup.md](./product-help-and-order-lookup.md) | Product help & how-to | binance (CLI, optional), spot / algo / derivatives-* (order lookup), skill-creator, skill-vetter |
| [token-deployment-and-onchain-pay.md](./token-deployment-and-onchain-pay.md) | Token launch & on-chain pay | onchain-pay-open-api, query-token-info, query-token-audit, query-address-info |
| [campaigns-and-rewards.md](./campaigns-and-rewards.md) | Campaigns & incentives | alpha (context) |
| [binance-square-posting.md](./binance-square-posting.md) | Content & social (Square) | square-post |
| [education-and-learning.md](./education-and-learning.md) | Education & learning | query-token-info (examples), skill-creator (optional) |
| [simple-earn-and-vip-loan.md](./simple-earn-and-vip-loan.md) | Earn & borrow (supplement) | simple-earn, vip-loan, assets |
| [fuzzy-intent-and-account-onboarding.md](./fuzzy-intent-and-account-onboarding.md) | Vague / onboarding intents | assets, p2p, fiat, spot (guided first steps) |

**Notes**

- **`weather`**: standalone tool skill; no separate Task doc.
- **Environment safety**: pair with **`healthcheck`** via [api-authorization-and-debugging.md](./api-authorization-and-debugging.md).
- **On-chain tasks**: if there is a **contract address**, **audit first** (`query-token-audit`)—see [onchain-signals-and-security.md](./onchain-signals-and-security.md) §A.

---

## Skill names (aligned with binance-skills-hub)

**26** skills ship in this hub (`name` in each `SKILL.md`). **3** names appear in Tasks but are **not** in this repo: `healthcheck`, `skill-creator`, `weather` (deployment-dependent).

**CLI bundle (spot / convert / USDS-M)**: `binance` — `binance-cli` vs REST `spot` / `convert` / `derivatives-trading-usds-futures`. Entry: **`../binance/binance/SKILL.md`** (root: `skills/binance/binance/SKILL.md`); refs: `references/spot.md`, `convert.md`, `futures-usds.md`, `auth.md`.

**Spot & trading (REST)**: `spot`, `convert`, `algo`, `p2p`

**Derivatives**: `derivatives-trading-usds-futures`, `derivatives-trading-coin-futures`, `derivatives-trading-options`, `derivatives-trading-portfolio-margin`, `derivatives-trading-portfolio-margin-pro`, `margin-trading`

**Assets & earn**: `assets`, `simple-earn`, `vip-loan`, `fiat`

**Market (`../binance-web3/`)**: `query-token-info`, `crypto-market-rank`, `binance-tokenized-securities-info`, `trading-signal`, `meme-rush`

**Security**: `query-token-audit`, `query-address-info`; **not in hub**: `skill-vetter`, `healthcheck`

**Social & pay**: `square-post`, `onchain-pay-open-api` (folder `onchain-pay`), `alpha`

**Account & tools**: `sub-account`; **not in hub**: `skill-creator`, `weather`

---

## Compliance

- Tasks describe **routing and capability**, not investment advice; for trades, loans, and campaign rules, follow **Binance official pages and product terms**.
- Campaigns: **no** full campaign-text API in this skill set—rules live on the website/app.

---

## Maintenance

- Update **`intent_category.json`** when intents change, then add or remap Task `*.md` files.
- Keep §B endpoint lists consistent with the corresponding hub **`SKILL.md`** under `../binance/` and `../binance-web3/`.
