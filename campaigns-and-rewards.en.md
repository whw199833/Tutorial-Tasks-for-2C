# Campaigns and Rewards

## Description

**Task**: Platform campaigns, promos, Beta incentives, campaign outcome discussion; Skills **do not** guarantee live campaign page text; use `alpha` and related token capabilities as supplement; rules depend on website, announcements, and Agent knowledge freshness.

**Typical intents**: recent Binance campaigns and promos; Beta incentives; campaign results and naming ideas (social).

---

## Recommended Skills

| Role | Skill | Use |
|------|--------|-----|
| Context | `alpha` | Alpha-related tokens and campaign context if applicable |
| General | `query-token-info` / `crypto-market-rank` | Quotes when a specific coin is named |
| Boundary | — | Rules follow official pages |

---

## Plan

> Aligned with `Task_upgrade_advice.en.md` §9: **official rules first** → optional quotes for “coins mentioned” → optional user self-check volume; discussion = no API.

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Default** | Campaign/Beta terms → text + point to App/site; **no** standalone campaign full-text API. |
| **Optional: token context** | Campaign names a coin: `alpha` token list or `unified/rank` (`rankType=20`) or `search/ai`, only to show product context. |
| **Optional: task progress** | User self-check volume: `getUserAsset` + `openOrders`/`allOrders` (window) → **not guaranteed** to match campaign stats. |
| **Discussion** | Naming ideas, etc. → zero REST. |

### B. API quick reference

1. **Disclaimer**: campaign/Beta rules have **no** dedicated campaign API in this Skill set; official site and App are authoritative.

2. **If campaign binds a token quote** (optional)
   - `alpha`: `GET https://www.binance.com/bapi/defi/v1/public/wallet-direct/buw/wallet/cex/alpha/all/token/list`
   - Web3: `POST .../unified/rank/list/ai` (`rankType=20` Alpha board) or `query-token-info` `search/ai`.

3. **If campaign involves trading tasks**
   - Balance: `assets` `POST /sapi/v3/asset/getUserAsset`; orders: see `trading-execution.en.md` `/api/v3/order`, `/fapi/v1/openOrders`, etc.

4. **Discussion**: no REST.

---

## Usage

### Structured notes

- **APIs cannot replace campaign terms**: quotes only support price / list presence, not reward eligibility.
- **Default zero requests**: answer rules and links first, then decide whether to pull quotes.

### Boundary

- **Light calls**: most scenarios need no API.
