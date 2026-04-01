# Token Deployment and On-chain Pay

## Description

**Task**: Create, launch, deploy tokens on BNB Chain and similar, plus social link integration. Binance Skills focus on **trading, assets, quotes, audit**—**no full contract IDE or one-click deploy**; use Skills for payments, token info, and security checks.

**Typical intents**: create and launch on BNB Chain; delegated deploy and naming; integrate social links.

---

## Recommended Skills

| Role | Skill | Use |
|------|--------|-----|
| Pay / buy | `onchain-pay-open-api` | Fiat on-ramp or send to on-chain address where applicable |
| Info | `query-token-info` | Reference for existing tokens |
| Safety | `query-token-audit` | Pre/post deploy risk awareness |
| Address | `query-address-info` | Deployer wallet positions |

---

## Plan

> Aligned with `Task_upgrade_advice.en.md` §8: no one-click deploy REST; pay flow **pairs → payment → quote → address/network → pre-order → poll**; post-deploy **search + audit + address positions**.

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Need** | Fiat to address vs move existing coins → set `pre-order` `customization` (e.g. `SEND_PRIMARY`). |
| **Capability** | `trading-pairs` → `payment-method-list` → `estimated-quote` (same fiat/crypto/amount basis). |
| **Pre-order** | Required/configured **`address` + `network`**; `crypto-network` for fees and limits. |
| **Order** | `pre-order` → `order` poll to terminal state. |
| **Post-deploy** | If contract: `search` + `audit` + `query-address-info` (deployer wallet); **does not** replace Remix/Hardhat. |

### B. API quick reference

1. **Onchain Pay (`onchain-pay-open-api`, RSA signing, see Skill `scripts/sign_and_call.sh`)**
   - `papi/v1/ramp/connect/buy/trading-pairs`: supported fiat/pairs.
   - `papi/v1/ramp/connect/buy/payment-method-list` (v2: `papi/v2/...`): methods and limits.
   - `papi/v1/ramp/connect/buy/estimated-quote`: `fiatCurrency`, `requestedAmount`, `payMethodCode`, `amountType`, etc.
   - `papi/v1/ramp/connect/gray/buy/pre-order`: `externalOrderId`, `merchantCode`, `merchantName`, `ts`, plus `address`, `network`, `customization` (`SEND_PRIMARY`, `ON_CHAIN_PROXY_MODE`, …).
   - `papi/v1/ramp/connect/order`: query by `externalOrderId`.
   - `papi/v1/ramp/connect/crypto-network`: chains, fees, limits.

2. **Post-deploy token info (`query-token-info`)**
   - `GET https://web3.binance.com/bapi/defi/v5/public/wallet-direct/buw/wallet/market/token/search/ai?keyword=` (contract address).

3. **Safety (`query-token-audit`)**
   - `POST https://web3.binance.com/bapi/defi/v1/public/wallet-direct/security/token/audit` (`binanceChainId`, `contractAddress`, `requestId`).

4. **Deployer positions (`query-address-info`)**
   - `GET https://web3.binance.com/bapi/defi/v3/public/wallet-direct/buw/wallet/address/pnl/active-position-list/ai?address=&chainId=&offset=`.

5. **Contract authoring/deploy**: no dedicated Binance REST; Remix/Hardhat etc. are out of this Skill set.

---

## Usage

### Structured notes

- **Deploy vs Onchain Pay split**: compile/deploy not in this API flow; Pay only moves funds to on-chain addresses.

### Credentials

- **Onchain Pay**: `BASE_URL`, `CLIENT_ID`, `API_KEY`, RSA PEM; `pre-order` usually needs `address` + `network` (or `.local.md` defaults).
- **With `onchain-signals-and-security.en.md`**: after launch, keep using `query-token-audit` and `query-token-info`.
