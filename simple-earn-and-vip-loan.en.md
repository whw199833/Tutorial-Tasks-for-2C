# Simple Earn and VIP Loan

## Description

**Task summary**: Flexible/fixed subscribe & redeem, Simple Earn product browse/ops, VIP Loan Q&A and in-scope queries—distinct from plain **spot balance**; focuses on **yield and borrow** product paths.

**Typical user phrasing**: subscribe/redeem, APY, fixed terms, borrow rates, VIP loan limits and steps (may overlap account-asset intents—this task stresses **product type**).

---

## Recommended skill mix

| Role | Skill | Use |
|------|--------|-----|
| Primary | `simple-earn` | Simple Earn (flexible/fixed, etc.) |
| Primary | `vip-loan` | VIP loan |
| Secondary | `assets` | Wallet view before/after moving to/from earn |
| Secondary | `margin-trading` | Clarify “borrow” vs **margin borrow** |

**binance-skills-hub**: `simple-earn`, `vip-loan`, `assets`, `margin-trading` → **`skills/binance/<skill>/SKILL.md`** (REST; no `binance-cli` overlap).

---

## Plan

> Aligns with `Task_upgrade_advice.en.md` §12: **pick product line** (Simple Earn / VIP Loan / margin) → list → positions → preview → subscribe/borrow → `getUserAsset` check.

### Status checks and when you cannot proceed

- **Before planning**: Product line clear; **available balance** for subscribe/stake/collateral; **existing positions/orders** (duplicate subscribe?, borrow limit?).
- **If preview fails or balance short**: (1) Product response and gap; (2) Transfer first?, other product/term?; (3) **No** `subscribe`/`borrow` until user confirms preview and amount.
- **Cross-task rules**: [Task_upgrade_advice.en.md](./Task_upgrade_advice.en.md).

### A. Structured pipeline (DAG)

**Step 0: Prerequisite state check — *MANDATORY***

---

| Step | Action |
|------|--------|
| **Product line** | Retail earn → `simple-earn`; VIP collateralized loan → `vip-loan`; spot margin borrow/repay → `margin-trading` (different product). |
| **Simple Earn** | `simple-earn/account` → `flexible/list` or `locked/list` → `subscriptionPreview` → confirm → `subscribe`; redeem: `position` → `redeem`. |
| **VIP Loan** | `loanable/data` + `collateral/data` + `interestRate` → confirm → `borrow`; ongoing `ongoing/orders`, repay `repay` / renew `renew`. |
| **Balance check** | Before/after `getUserAsset` (+ `get-funding-asset`) align with `account-and-asset-management.en.md`. |

### B. Endpoint quick reference

**Base**: `https://api.binance.com` (signed USER_DATA unless noted).

1. **`simple-earn`**: `GET /sapi/v1/simple-earn/account`; flexible/locked list, position, preview, subscribe, redeem; BFUSD/RWUSD modules per SKILL.
2. **`vip-loan`**: interest, loanable, collateral, `borrow`, `repay`, `renew`, `ongoing/orders` per SKILL.
3. **`assets`**: `POST /sapi/v3/asset/getUserAsset`, `POST /sapi/v1/asset/get-funding-asset` around earn moves.
4. **`margin-trading`**: `POST /sapi/v1/margin/borrow-repay`—not VIP loan.

---

## Usage guide

- **Split products first**: do not explain Simple Earn with VIP loan fields.
- **Balance before and after** subscribe/borrow for user-visible delta.
- **Retail earn** `simple-earn`; **VIP pledged loan** `vip-loan`; **spot margin** `margin-trading` `/sapi/v1/margin/*`.
- **With `account-and-asset-management.en.md`**: flexible/fixed **positions** → `simple-earn/position`; generic balance → `getUserAsset`.
