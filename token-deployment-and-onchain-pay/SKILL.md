---
name: token-deployment-and-onchain-pay
description: |
  Token launch/pay flows: on-chain pay (fiat→crypto / send to address), token info, audit, deployer address context—no full contract IDE in skill set.

  Typical intents: BNB Chain token create/launch; delegated deploy and naming; integrate social links.
metadata:
  author: binance-bigdata-team
  version: "1.0"
---

# Token Deployment and On-chain Pay

## Overview

| API | Function | Use Case |
|-----|----------|----------|
| Onchain Pay | `onchain-pay-open-api` | Pairs → quote → pre-order → order to address |
| Post-deploy | `query-token-info`, `query-token-audit` | Search contract + risk check |
| Deployer wallet | `query-address-info` | Holdings / PnL context |

## Description

**Task summary**: Create/launch/deploy tokens on BNB Chain etc. and social link integration. Binance skills focus on **trading, assets, market data, audit**—**no full contract IDE or one-click deploy**; use skills for **pay flows**, token info, and security checks.

**Typical intents**: BNB Chain token create/launch; delegated deploy and naming; integrate social links.

---

## Recommended skill mix

| Role | Skill | Use |
|------|--------|-----|
| Pay / buy | `onchain-pay-open-api` | Fiat-to-crypto or send to on-chain address when flow applies |
| Info | `query-token-info` | Reference for existing tokens |
| Security | `query-token-audit` | Pre/post deploy risk awareness |
| Address | `query-address-info` | Deployer wallet holdings |

**binance-skills-hub**: `onchain-pay-open-api` → **`skills/binance/onchain-pay/SKILL.md`** (`name` field); other Web3 → **`skills/binance-web3/`**.

---

## Plan

> Aligns with `Task_upgrade_advice.md` §8: no one-click deploy REST; pay **pairs → payment → quote → address/network → pre-order → poll**; post-deploy **search + audit + address holdings**.

### Status checks and when you cannot proceed

- **Before planning**: Pay path **fiat/pair, amount, recipient address and network**; before `estimated-quote`, enough funds (optional `assets`).
- **If on-chain pay fails or limit hit**: (1) Error and limits; (2) Change method/network or amount?; (3) User must confirm address/network before final `pre-order`/`order`.
- **Cross-task rules**: [Task_upgrade_advice.md](./Task_upgrade_advice.md).

### A. Structured pipeline (DAG)

**Step 0: Prerequisite state check — *MANDATORY*** when user ties to balances or trading.

---

| Step | Action |
|------|--------|
| **Need** | Fiat buy to address vs send existing crypto → `pre-order` `customization` (e.g. `SEND_PRIMARY`). |
| **Capability** | `trading-pairs` → `payment-method-list` → `estimated-quote` (consistent fiat/crypto/amount). |
| **Pre-order** | **`address` + `network`** required; `crypto-network` for fees/limits. |
| **Order** | `pre-order` → `order` poll to terminal state. |
| **Post-deploy** | If contract: `search` + `audit` + `query-address-info` (deployer)—**not** Remix/Hardhat replacement. |

### B. Endpoint quick reference

1. **Onchain Pay** (`onchain-pay-open-api`, RSA signing—see `scripts/sign_and_call.sh`): `trading-pairs`, `payment-method-list`, `estimated-quote`, `pre-order`, `order`, `crypto-network` per SKILL (`papi/v1` or `v2` paths).
2. **Post-deploy token**: `query-token-info` `search/ai` by contract address.
3. **Audit**: `POST .../security/token/audit`.
4. **Deployer holdings**: `GET .../address/pnl/active-position-list/ai`.
5. **Contract author/deploy**: no dedicated Binance REST; Remix/Hardhat off-platform.

---

## Usage guide

- **Deploy vs Pay**: compile/deploy not in this API set; Pay moves funds to on-chain addresses.
- **Credentials**: `BASE_URL`, `CLIENT_ID`, `API_KEY`, RSA PEM; `pre-order` usually needs `address` + `network`.
- **With `onchain-signals-and-security.md`**: new tokens—ongoing `query-token-audit` + `query-token-info`.
