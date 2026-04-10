# Tutorial-Tasks-for-Skills

For **Binance AI Pro (Clawbot)**-style scenarios: map online user intents to **Tasks**, and for each Task list **applicable Skills**, a **structured execution order (DAG)**, and **REST/endpoint quick reference**, so agents pick skills step-by-step instead of flat API calls.

---

## What’s in this repo

| Item | Description |
|------|-------------|
| **Task docs** (`*.cn.md` / `*.en.md`) | One file per intent: `Description`, recommended Skills, **Plan (§A DAG + §B endpoint quick ref [+ §C `binance-cli` where applicable])**, usage notes. |
| **[Task_upgrade_advice.en.md](./Task_upgrade_advice.en.md)** | Cross-task **default execution DAG** (aligned with §A in each file); endpoint detail still comes from each Task §B and `binance-skills-hub`. |
| **`binance-skills-hub`** | `skills/binance/`: exchange REST `SKILL.md`; **`skills/binance/binance`** is the **`binance-cli`** entry (`spot` / `convert` / `futures-usds`, mapped to REST). `skills/binance-web3/`: market, audit, signals, etc. |
| **Intent taxonomy** | Categories and example queries: `intent_category.json`. |

---

## How to read (recommended)

1. **Pick intent**: Find the category in `intent_category.json`, then map to the **Task file** in the table below.
2. **Pre-planning account check** (when funds, trades, transfers, earn subscribe, or borrow are involved): Before **planning** for the user, confirm current account state (available balance, open and in-flight orders, locks/freeze)—see the “Pre-planning” section in [Task_upgrade_advice.en.md](./Task_upgrade_advice.en.md); pure market data, education, or compliance explainers may skip.
3. **Read the DAG**: Open the Task, start with **Plan → §A**, to see what narrows first.
4. **Look up endpoints**: Task **§B** matches `binance-skills-hub` `SKILL.md`; for **spot / convert / USDS-M** with `binance-cli` installed, also see the **`binance`** skill (`skills/binance/binance/SKILL.md`) and `references/*.md`.

---

## Task index (intent → file → core skills)

| Task file | Intent category (intent_category) | Core skills |
|-----------|-----------------------------------|-------------|
| [account-and-asset-management.en.md](./account-and-asset-management.en.md) | Account & assets | assets, sub-account, fiat |
| [api-authorization-and-debugging.en.md](./api-authorization-and-debugging.en.md) | API & system auth | assets, derivatives-*, sub-account, healthcheck, skill-vetter |
| [trading-execution.en.md](./trading-execution.en.md) | Trading strategy & execution | **binance** (CLI), spot, convert, algo, p2p, margin-trading, derivatives-* |
| [market-data-and-analysis.en.md](./market-data-and-analysis.en.md) | Market analysis & quotes | query-token-info, crypto-market-rank, trading-signal, meme-rush, binance-tokenized-securities-info |
| [onchain-signals-and-security.en.md](./onchain-signals-and-security.en.md) | On-chain data & smart money | trading-signal, query-token-audit, query-address-info, query-token-info, meme-rush |
| [token-research-and-opportunities.en.md](./token-research-and-opportunities.en.md) | Token research & opportunities | query-token-info, crypto-market-rank, trading-signal, query-token-audit, meme-rush, alpha |
| [product-help-and-order-lookup.en.md](./product-help-and-order-lookup.en.md) | Product help & how-to | binance (CLI, optional), spot / algo / derivatives-* (order lookup), skill-creator, skill-vetter |
| [token-deployment-and-onchain-pay.en.md](./token-deployment-and-onchain-pay.en.md) | Token launch & on-chain pay | onchain-pay-open-api, query-token-info, query-token-audit, query-address-info |
| [campaigns-and-rewards.en.md](./campaigns-and-rewards.en.md) | Campaigns & incentives | alpha (context) |
| [binance-square-posting.en.md](./binance-square-posting.en.md) | Content & social (Square) | square-post |
| [education-and-learning.en.md](./education-and-learning.en.md) | Education & learning | query-token-info (examples), skill-creator (optional) |
| [simple-earn-and-vip-loan.en.md](./simple-earn-and-vip-loan.en.md) | Earn & borrow (supplement) | simple-earn, vip-loan, assets |
| [fuzzy-intent-and-account-onboarding.en.md](./fuzzy-intent-and-account-onboarding.en.md) | Vague / onboarding intents | assets, p2p, fiat, spot (guided first steps) |

**Notes**:

- `weather` is a standalone tool skill—call as needed; no separate Task doc.
- For “environment safety”, pair with `healthcheck` from [api-authorization-and-debugging.en.md](./api-authorization-and-debugging.en.md).
- On-chain/monitor tasks: **if there is a contract address, audit first** (`query-token-audit`)—see [onchain-signals-and-security.en.md](./onchain-signals-and-security.en.md) §A.

---

## Skill names (aligned with binance-skills-hub)

**26** skills in `binance-skills-hub` (`name` in each `SKILL.md`); **3** referenced in Tasks but **not** in this hub: `healthcheck`, `skill-creator`, `weather` (deployment-dependent).

**CLI bundle (spot / convert / USDS-M)**: `binance` — `binance-cli` subcommands vs `spot` / `convert` / `derivatives-trading-usds-futures` REST; entry **`skills/binance/binance/SKILL.md`**, refs `references/spot.md`, `convert.md`, `futures-usds.md`, `auth.md`.

**Spot & trading (REST)**: `spot`, `convert`, `algo`, `p2p`

**Derivatives**: `derivatives-trading-usds-futures`, `derivatives-trading-coin-futures`, `derivatives-trading-options`, `derivatives-trading-portfolio-margin`, `derivatives-trading-portfolio-margin-pro`, `margin-trading`

**Assets & earn**: `assets`, `simple-earn`, `vip-loan`, `fiat`

**Market (binance-web3)**: `query-token-info`, `crypto-market-rank`, `binance-tokenized-securities-info`, `trading-signal`, `meme-rush`

**Security**: `query-token-audit`, `query-address-info`; **not hub**: `skill-vetter`, `healthcheck`

**Social & pay**: `square-post`, `onchain-pay-open-api` (folder `onchain-pay`), `alpha`

**Account & tools**: `sub-account`; **not hub**: `skill-creator`, `weather`

---

## Compliance

- Tasks describe **routing and capability**, not investment advice; for trades, loans, and campaign rules, follow **Binance official pages and product terms**.
- Campaigns ([campaigns-and-rewards.en.md](./campaigns-and-rewards.en.md)): **no** full campaign text API—rules live on website/app.

---

## Maintenance

- When adding or changing intents: update `intent_category.json`, then add or remap Task files as needed.

**Chinese versions**: see `README.md` and `*.cn.md` in this repository.
