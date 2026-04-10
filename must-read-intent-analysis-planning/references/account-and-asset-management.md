# Account and Asset Management

## Overview

| API | Function | Use Case |
|-----|----------|----------|
| Wallet balances | `getUserAsset`, wallet summary | Spot and funding snapshot |
| Ledger trace | Deposit / withdraw / universal transfer history | Reconcile “where did funds go” |
| Sub-account | List, assets, `universalTransfer` | Master–sub scope and moves |
| Fiat rails | Fiat SAPI orders + public BAPI capabilities | Fiat channels and account orders |

## Description

**Task summary**: Help users query main- and sub-account assets, reconcile balances and distribution, track deposit/withdrawal/transfer status, and support sub-account–related questions and operational guidance.

**Typical intents** (from intent taxonomy): Binance account assets and trading status; main-account balance confirmation; delegated-operation authorization and fund checks; balance and asset distribution; fund flow tracing; account opening and trading questions; account feature closure and refund inquiries (informational).

**Skill boundaries**: Assets and transfers → **`assets`**; sub-accounts → **`sub-account`**; fiat rails → **`fiat`** as secondary. If the user also asks about contracts or unified margin, bridge to derivative skills in **[trading-execution.md](./trading-execution.md)**.

---

## Recommended skill mix

| Role | Skill | Use |
|------|--------|-----|
| Primary | `assets` | Balances, deposits/withdrawals, transfers, asset distribution |
| Primary | `sub-account` | Sub-account list, permissions, master–sub transfer context |
| Secondary | `fiat` | Fiat deposit/withdraw order reconciliation |
| Bridge | `derivatives-trading-portfolio-margin` / `derivatives-trading-portfolio-margin-pro` | Only when the user clearly asks for unified / portfolio margin view |

---

## Plan

> Aligns with `task-upgrade-advice.md` §1: scope → snapshot → ledger alignment → sub-account/fiat branches → consolidated output.

### Status checks and when you cannot proceed

- **Before planning**: Confirm scope (main vs funding vs sub-account vs fiat) and whether the **time window** for the issue is clear.
- **If ledgers disagree with snapshot or APIs lack permission**: (1) State what was checked and what is missing; (2) Ask whether other wallets/sub-accounts exist, or whether the user can provide a txId or in-app order time; (3) Do **not** invent “where funds went”; for policy/account-closure topics, point to the website and support.
- **Cross-task rules**: See the opening of [task-upgrade-advice.md](./task-upgrade-advice.md) for global flow and “ask the user” guidance.

### A. Structured pipeline (DAG)

**Step 0: Prerequisite state check — *MANDATORY***

> **Goal**: Before concrete steps, understand the user’s current state for safer, personalized guidance.
> **Core skills**: `assets`, `spot` (for `getOrders`), `derivatives-trading-usds-futures` (for `getPositions`)

1. **Account assets**: Call `assets.getUserAssets`; review available and total value per wallet (especially spot `SPOT` and funding `FUNDING`).
2. **Open positions**: Call `derivatives-trading-usds-futures.getPositions`; check USDS-M positions (side, size, unrealized PnL).
3. **Recent trades and open orders**: Call `spot.getOrders`; note trading habits (preferred pairs) and any open orders.

> **After diagnosis**: Adjust downstream steps. If the user already has related exposure, plan around it; if funds are insufficient, see **[fuzzy-intent-and-account-onboarding.md](./fuzzy-intent-and-account-onboarding.md)** for funding guidance.

---

| Step | Action |
|------|--------|
| **Scope** | Separate: main spot only / includes Funding / includes sub-account / includes fiat orders / includes contracts or PM (latter → [trading-execution.md](./trading-execution.md)). |
| **Balance snapshot** | `getUserAsset` or `wallet/balance` → answering “how much X left” may stop here; “where did money go” → ledgers. |
| **Ledger alignment** | In the time window: `deposit/hisrec`, `withdraw/history`, `asset/transfer` (GET) **cross-check** with snapshot (large moves should match records). |
| **Sub-account** | Only when sub-account email or master–sub transfer is explicit: `sub-account/list` → `assets` → `universalTransfer` history if needed. |
| **Fiat** | Channels/limits: public `get-capabilities` / `get-price`; **my orders**: SAPI `fiat/orders`, `fiat/payments` aligned with asset timeline. |
| **Close out** | Per wallet type: **balance + 3–5 key recent ledger lines**; refunds/closures—state facts only and cite the website. |

### B. Endpoint quick reference

HTTP paths and parameters follow the `assets`, `sub-account`, and `fiat` **SKILL.md** files under `binance-skills-hub/skills/binance/<skill>/`. Base: `https://api.binance.com` (REST needs `X-MBX-APIKEY` plus signed `timestamp`/`signature` unless marked public).

**Spot snapshot supplement**: If **`binance-cli`** is enabled, the `binance` skill ([`binance` CLI entry](../../../binance/binance/SKILL.md)) can run `binance-cli spot get-account` to **cross-check** REST `assets` / `spot` account views; deposits/withdrawals/transfers remain on **`assets`**.

1. **Main-account spot / funding balances**
   - `POST /sapi/v3/asset/getUserAsset`: per-asset balances (optional `asset` filter).
   - `GET /sapi/v1/asset/wallet/balance`: wallet summary by `quoteAsset`.
   - `GET /api/v3/account` (`spot` skill): spot balances and order locks (if tied to spot orders).

2. **Deposit / withdrawal / transfer tracing**
   - `GET /sapi/v1/capital/deposit/hisrec`: deposit history (`coin`, `status`, `startTime`/`endTime`, etc.).
   - `GET /sapi/v1/capital/withdraw/history`: withdrawal history.
   - `GET /sapi/v1/asset/transfer` (GET): universal transfer history; `POST /sapi/v1/asset/transfer`: transfer (`type`, `asset`, `amount`, `fromSymbol`/`toSymbol` per docs).

3. **Funding wallet / snapshots**
   - `POST /sapi/v1/asset/get-funding-asset`: funding wallet assets.
   - `GET /sapi/v1/accountSnapshot`: daily snapshots (`type`: SPOT/MARGIN, etc.).

4. **Sub-accounts (master view)**
   - `GET /sapi/v1/sub-account/list`: list (`email`, `page`, `limit`).
   - `GET /sapi/v4/sub-account/assets` or `GET /sapi/v3/sub-account/assets`: assets for a given sub-account `email`.
   - `GET /sapi/v1/sub-account/universalTransfer` (GET): master–sub history; `POST .../universalTransfer`: universal transfer (`fromAccountType`/`toAccountType`: `SPOT`, `USDT_FUTURE`, `COIN_FUTURE`, `MARGIN`, `ISOLATED_MARGIN`, etc.).

5. **Fiat orders and payments (API key required)**
   - `GET /sapi/v1/fiat/orders`: fiat order history (`transactionType`, etc.).
   - `GET /sapi/v1/fiat/payments`: fiat payment history.
   - **Public (no key)**: `https://www.binance.com/bapi/fiat/v1/public/fiatpayment/agent/get-capabilities?country={CC}` and same-prefix `get-buy-and-sell-payment-methods`, `get-deposit-and-withdraw-payment-methods`, `get-price` (capabilities/channels/reference price only—not account ledgers).

---

## Usage guide

### Structure

- **No skipping steps**: Do not fan out all SAPI calls before main vs sub is clear; sub-account flows only when the conversation is explicit.
- **Ledgers before guesses**: For “where did my money go,” align deposit/withdraw/transfer timelines before explaining.

### Auth and APIs

- **Auth**: `assets` / `sub-account` / signed fiat SAPI use `BINANCE_API_KEY`, `BINANCE_SECRET_KEY`; signing per each skill’s `references/authentication.md`; `User-Agent` per SKILL (e.g. `binance-wallet/1.1.0 (Skill)`).
- **REST assets before sub-account**: Main-account balance alone can use `getUserAsset`; call sub-account `list` / `assets` / `universalTransfer` only when email or transfers are in scope.
- **Public fiat BAPI vs SAPI**: Quotes/channels → `bapi/fiat/.../agent/*`; **account-level** fiat orders → `GET /sapi/v1/fiat/orders`, `/sapi/v1/fiat/payments`.
- **On-chain / mainnet**: Skills may require user `CONFIRM` before mainnet actions; delegated-operation disclaimers apply.
- **With [trading-execution.md](./trading-execution.md)**: Post-trade order lookup is not this task; spot orders use `spot` `/api/v3/order`, `/api/v3/openOrders`.
