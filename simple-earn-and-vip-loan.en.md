# Simple Earn and VIP Loan

## Description

**Task**: Flexible/fixed subscribe & redeem, Simple Earn product lookup and actions, and VIP Loan questions and in-scope queries—distinct from pure “spot balance,” focused on **yield and lending** product paths.

**Typical user phrases**: subscribe/redeem, APY, fixed term, borrow rates, VIP loan limits and operations (can overlap account-asset intent; this Task emphasizes product type).

---

## Recommended Skills

| Role | Skill | Use |
|------|--------|-----|
| Primary | `simple-earn` | Simple Earn (flexible/fixed) |
| Primary | `vip-loan` | VIP Loan |
| Secondary | `assets` | Wallet view before/after moving to/from earn |
| Secondary | `margin-trading` | Clarify vs “margin borrow” when user conflates products |

---

## Plan

> Aligned with `Task_upgrade_advice.en.md` §12: **pick product line** (Simple Earn / VIP Loan / spot margin) → lists → positions → preview → subscribe/borrow → `getUserAsset` check.

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Product line** | Retail earn → `simple-earn`; VIP collateralized loan → `vip-loan`; spot margin borrow/repay → `margin-trading` (different from VIP). |
| **Simple Earn** | `simple-earn/account` → `flexible/list` or `locked/list` → `subscriptionPreview` → confirm → `subscribe`; before redeem `position` → `redeem`. |
| **VIP Loan** | `loanable/data` + `collateral/data` + `interestRate` → confirm → `borrow`; live `ongoing/orders`, repay `repay`, renew `renew`. |
| **Funds check** | Before/after: `getUserAsset` (and `get-funding-asset`) aligned with `account-and-asset-management.en.md`. |

### B. API quick reference

**Base**: `https://api.binance.com` (signed USER_DATA unless noted).

1. **Simple Earn (`simple-earn`)**
   - Overview: `GET /sapi/v1/simple-earn/account`
   - Lists: `GET /sapi/v1/simple-earn/flexible/list`, `GET /sapi/v1/simple-earn/locked/list`
   - Positions: `GET /sapi/v1/simple-earn/flexible/position`, `GET /sapi/v1/simple-earn/locked/position`
   - Preview: `GET /sapi/v1/simple-earn/flexible/subscriptionPreview`, `GET /sapi/v1/simple-earn/locked/subscriptionPreview`
   - Subscribe: `POST /sapi/v1/simple-earn/flexible/subscribe`, `POST /sapi/v1/simple-earn/locked/subscribe`
   - Redeem: `POST /sapi/v1/simple-earn/flexible/redeem`, `POST /sapi/v1/simple-earn/locked/redeem`
   - BFUSD/RWUSD: see SKILL `/sapi/v1/bfusd/*`, `/sapi/v1/rwusd/*`

2. **VIP Loan (`vip-loan`)**
   - Rates & loanable: `GET /sapi/v1/loan/vip/request/interestRate?loanCoin=`, `GET /sapi/v1/loan/vip/loanable/data`
   - Collateral: `GET /sapi/v1/loan/vip/collateral/data`
   - Borrow: `POST /sapi/v1/loan/vip/borrow` (`loanAccountId`, `loanCoin`, `loanAmount`, `collateralAccountId`, `collateralCoin`, `isFlexibleRate`, …)
   - Repay/renew: `POST /sapi/v1/loan/vip/repay`, `POST /sapi/v1/loan/vip/renew`
   - Open orders: `GET /sapi/v1/loan/vip/ongoing/orders`

3. **Funds check (`assets`)**
   - `POST /sapi/v3/asset/getUserAsset`, `POST /sapi/v1/asset/get-funding-asset`: balances around earn moves.

4. **Margin confusion (`margin-trading`)**
   - Spot margin borrow/repay: `POST /sapi/v1/margin/borrow-repay`; different product line from VIP Loan.

---

## Usage

### Structured notes

- **Split products before APIs**: do not explain Simple Earn subscribe with VIP Loan responses (rates and risks differ).
- **Balance before and after**: helps show “delta” and consistency.

### Products & accounts

- **Routing**: retail earn → `simple-earn`; VIP-style collateral loan → `vip-loan`; spot margin → separate `/sapi/v1/margin/*` via `margin-trading`.
- **With `account-and-asset-management.en.md`**: flexible/fixed **positions** use `simple-earn/position`; generic balance still `getUserAsset`.
