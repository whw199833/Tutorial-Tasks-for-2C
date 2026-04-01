# Account and Asset Management

## Description

**Task**: Help users with main/sub-account balances, distribution checks, deposit/withdrawal/transfer status, and sub-account–related questions and assisted actions.

**Typical intents** (from intent taxonomy): query Binance assets and trade status; main-account balance; delegated trading authorization and funds review; balance and asset distribution; trace where funds went; onboarding and trading questions; account feature closure and refund–related questions (informational).

**Skill boundaries**: Assets and transfers use `assets`; sub-accounts use `sub-account`; fiat history uses `fiat` as needed; if the user also asks about futures / unified margin, bridge to derivatives Skills in `trading-execution.en.md`.

---

## Recommended Skills

| Role | Skill | Use |
|------|--------|-----|
| Primary | `assets` | Balances, deposits/withdrawals, transfers, distribution |
| Primary | `sub-account` | Sub-account list, permissions, transfer relationships |
| Secondary | `fiat` | Fiat deposit/withdraw record checks |
| Bridge | `derivatives-trading-portfolio-margin` / `derivatives-trading-portfolio-margin-pro` | Only when the user explicitly asks unified / portfolio margin views |

---

## Plan

> Aligned with `Task_upgrade_advice.en.md` §1: scope → snapshot → reconcile flows → sub-account/fiat branches → concise output.

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Scope** | Separate: spot main only / includes Funding / includes sub-accounts / includes fiat orders / includes futures or PM (last case → `trading-execution.en.md`). |
| **Balance snapshot** | `getUserAsset` or `wallet/balance` → “how much X” can end here; “where did money go” → go to history. |
| **Reconcile history** | Within a time window: `deposit/hisrec`, `withdraw/history`, `asset/transfer` (GET) **cross-check** against snapshot (large moves should match records). |
| **Sub-accounts** | Only when sub-account email or main↔sub transfer is explicit: `sub-account/list` → `assets` → `universalTransfer` history if needed. |
| **Fiat** | Channels/limits: Public `get-capabilities` / `get-price`; **my orders**: SAPI `fiat/orders`, `fiat/payments` aligned with asset timeline. |
| **Output** | By wallet type: **balance + 3–5 recent key movements**; refunds / closure policy: state facts only and point to official site. |

### B. API quick reference

HTTP paths and parameters follow Skill `assets`, `sub-account`, `fiat` `SKILL.md` in `binance-skills-hub`. Base: `https://api.binance.com` (REST needs `X-MBX-APIKEY` + signed `timestamp`/`signature` unless marked Public).

1. **Main spot / funding balances**
   - `POST /sapi/v3/asset/getUserAsset`: per-asset balances (optional `asset` filter).
   - `GET /sapi/v1/asset/wallet/balance`: wallet view by `quoteAsset`.
   - `GET /api/v3/account` (`spot` Skill): spot balances and open-order locks (if tying to spot orders).

2. **Deposit / withdrawal / transfer tracking**
   - `GET /sapi/v1/capital/deposit/hisrec`: deposit history (`coin`, `status`, `startTime`/`endTime`, etc.).
   - `GET /sapi/v1/capital/withdraw/history`: withdrawal history.
   - `GET /sapi/v1/asset/transfer` (GET): Universal Transfer history; `POST /sapi/v1/asset/transfer`: transfer (`type`, `asset`, `amount`, `fromSymbol`/`toSymbol` per docs).

3. **Funding wallet / snapshots**
   - `POST /sapi/v1/asset/get-funding-asset`: Funding Wallet assets.
   - `GET /sapi/v1/accountSnapshot`: daily snapshot (`type`: SPOT/MARGIN, etc.).

4. **Sub-accounts (main-account view)**
   - `GET /sapi/v1/sub-account/list`: list (`email`, `page`, `limit`).
   - `GET /sapi/v4/sub-account/assets` or `GET /sapi/v3/sub-account/assets`: assets for a given sub-account `email`.
   - `GET /sapi/v1/sub-account/universalTransfer` (GET): main↔sub history; `POST .../universalTransfer`: Universal Transfer (`fromAccountType`/`toAccountType`: `SPOT`, `USDT_FUTURE`, `COIN_FUTURE`, `MARGIN`, `ISOLATED_MARGIN`, etc.).

5. **Fiat orders and payments (API key required)**
   - `GET /sapi/v1/fiat/orders`: fiat order history (`transactionType`, etc.).
   - `GET /sapi/v1/fiat/payments`: fiat payment flow.
   - **Public capability (no key)**: `https://www.binance.com/bapi/fiat/v1/public/fiatpayment/agent/get-capabilities?country={CC}`, and same-prefix `get-buy-and-sell-payment-methods`, `get-deposit-and-withdraw-payment-methods`, `get-price` (capabilities/reference only, not account statements).

---

## Usage

### Structured notes

- **Do not skip steps**: do not fan out all SAPIs before clarifying main vs sub; sub-account APIs only after the conversation clearly involves them.
- **History over guesses**: when the user asks where funds went, align timeline with deposit/withdraw/transfer before explaining.

### Auth & APIs

- **Auth**: `assets` / `sub-account` / SAPI fiat endpoints use `BINANCE_API_KEY`, `BINANCE_SECRET_KEY`; signing per each Skill `references/authentication.md`; `User-Agent` per SKILL (e.g. `binance-wallet/1.1.0 (Skill)`).
- **REST assets before sub-account**: main-account balance alone can use `getUserAsset`; call `sub-account` `list` / `assets` / `universalTransfer` only when sub-account email or transfers appear.
- **Public fiat vs SAPI**: quotes/channels use `bapi/fiat/.../agent/*`; **account-level** fiat orders use `GET /sapi/v1/fiat/orders`, `/sapi/v1/fiat/payments`.
- **Mainnet actions**: Skill may require user to type `CONFIRM` before mainnet txs; assisted actions need compliance wording.
- **With `trading-execution.en.md`**: post-trade order lookup is not this Task; spot orders use `spot` `/api/v3/order`, `/api/v3/openOrders`.
