# Campaigns and Rewards

## Description

**Task summary**: Platform promos, campaigns, Beta incentives, campaign-result discussion; skills **do not** guarantee live campaign page text—use `alpha` and related token APIs as **supplement**; rules rely on website, announcements, and agent KB freshness.

**Typical intents**: Recent Binance campaigns/promos; Beta incentives; campaign results and naming/social ideas.

---

## Recommended skill mix

| Role | Skill | Use |
|------|--------|-----|
| Context | `alpha` | Alpha-token/campaign context when applicable |
| General | `query-token-info` / `crypto-market-rank` | Price context for named campaign tokens |
| Boundary | — | Rules follow official pages |

**binance-skills-hub**: `alpha` → **`skills/binance/alpha/SKILL.md`**; market supplements → **`skills/binance-web3/`**.

---

## Plan

> Aligns with `Task_upgrade_advice.en.md` §9: **rules from website** → optional token price proof → optional user self-check volume; discussion has no API.

### Status checks and when you cannot proceed

- **Before planning**: Rules from official site; “am I qualified?” → optional `getUserAsset` + order window with **disclaimer** that stats may not match campaign accounting.
- **If API cannot match task progress**: (1) API limits; (2) Ask user to check in-app campaign page; (3) **Do not** invent completed/incomplete tasks.
- **Cross-task rules**: [Task_upgrade_advice.en.md](./Task_upgrade_advice.en.md).

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Default** | Campaign/Beta terms → text + link to app/site; **no** full campaign API in this skill set. |
| **Optional: token context** | Named coin: `alpha` list or `unified/rank` (`rankType=20`) or `search/ai`—shows product context only. |
| **Optional: task progress** | User self-check volume: `getUserAsset` + `openOrders`/`allOrders` (window)—**not guaranteed** to match campaign stats. |
| **Discussion** | Naming ideas etc. → zero REST. |

### B. Endpoint quick reference

1. **Statement**: no standalone “campaign full text” API here—website + app authoritative.
2. **Token price (optional)**: `alpha` token list; Web3 `unified/rank/list/ai` or `query-token-info` `search/ai`.
3. **Trading-task style**: balances `POST /sapi/v3/asset/getUserAsset`; orders per `trading-execution.en.md`.
4. **Discussion**: no REST.

---

## Usage guide

- **APIs cannot replace terms**: price proves ticker/in-list, not reward eligibility.
- **Default zero requests**: answer rules + links first, then maybe market data.
