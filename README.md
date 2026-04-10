# Must-read: user intent analysis & planning

This package maps **Binance AI Pro (Clawbot)**–style user intents to **hub skills**, a **structured plan (DAG)**, and **endpoint quick references**. It does not replace executable skills; it tells agents **which skill to use and in what order**.

Executable skills live next to this folder: **`../binance/`** (REST, Simple Earn, `binance-cli` bundle under `binance/binance/`) and **`../binance-web3/`** (market, audit, signals).

---

## Layout

| Item | Path (from this `README`) |
|------|---------------------------|
| Entry skill (index + intent table) | [`must-read-user-intent-analysis-planning/SKILL.md`](./must-read-user-intent-analysis-planning/SKILL.md) |
| Cross-task pre-planning & DAG index | [`must-read-user-intent-analysis-planning/references/task-upgrade-advice.md`](./must-read-user-intent-analysis-planning/references/task-upgrade-advice.md) |
| Per-intent plans (§A DAG, §B/C endpoints) | [`must-read-user-intent-analysis-planning/references/`](./must-read-user-intent-analysis-planning/references/) |
| Intent taxonomy (English) | [`must-read-user-intent-analysis-planning/intent_category.json`](./must-read-user-intent-analysis-planning/intent_category.json) |

The inner directory name matches the skill folder convention (same name as the skill); start from **`SKILL.md`** inside it.

---

## How to use

1. **Classify intent** — Open [`intent_category.json`](./must-read-user-intent-analysis-planning/intent_category.json) for categories and example queries; match the user to one intent doc in the skill table.
2. **Pre-plan** — Before orders, transfers, earn subscribe, or borrow, read **Pre-planning** in [`task-upgrade-advice.md`](./must-read-user-intent-analysis-planning/references/task-upgrade-advice.md).
3. **Execute the plan** — Open the corresponding file under **`references/`** for status checks, §A pipeline, and §B (§C for CLI where applicable).
4. **Call APIs** — Follow links from those docs to **`../binance/<skill>/SKILL.md`**, **`../binance/binance/SKILL.md`** (CLI), and **`../binance-web3/<skill>/SKILL.md`**.

---

## Compliance

- Content here is **routing and capability**, not investment advice. Trades, loans, and campaign rules follow **Binance official product pages and terms**.

---

## Maintenance

- Update **`references/*.md`** and **`intent_category.json`** beside the inner **`SKILL.md`**; keep §B endpoint lists aligned with the matching hub **`SKILL.md`** under `binance/` and `binance-web3/`.
