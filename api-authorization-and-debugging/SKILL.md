---
name: api-authorization-and-debugging
description: |
  For API key setup, delegated trading auth, signature failures, sub-account API and futures order issues—connectivity and permission problems—help determine whether the issue is account permissions, key configuration, or client-side calls; within automation limits, pair with balance/position checks.

  Typical intents: Authorize AI to trade with profit targets; API key setup guidance; API config and call failures; sub-account BNB futures with API issues; account auth and signature debugging; trade command failures and signatures; query positions and troubleshoot API auth.
metadata:
  author: binance-bigdata-team
  version: "1.0"
---

# API Authorization and Debugging

## Overview

| API | Function | Use Case |
|-----|----------|----------|
| Key & account status | `apiRestrictions`, `apiTradingStatus`, `account/status` | Permission and trading-switch truth |
| Read paths | Spot / USDS-M / COIN-M smoke reads | Isolate which market layer fails |
| Sub-account + IP | Sub-account list, `ipRestriction` | Sub-key vs IP allowlist alignment |
| Environment | `healthcheck`, `skill-vetter` | Local hardening vs exchange errors |

## Description

**Task summary**: For API key setup, delegated trading auth, signature failures, sub-account API and futures order issues—**connectivity and permission** problems—help determine whether the issue is account permissions, key configuration, or client-side calls; within automation limits, pair with balance/position checks.

**Typical intents**: Authorize AI to trade with profit targets; API key setup guidance; API config and call failures; sub-account BNB futures with API issues; account auth and signature debugging; trade command failures and signatures; query positions and troubleshoot API auth.

**Skill boundaries**: No single skill “generates or fixes” user-local code; `healthcheck` is environment/system hardening; `assets` / derivative skills **verify** whether auth works (read balances/positions/order error semantics). `skill-vetter` reviews third-party skill install risk—different from exchange API debugging.

---

## Recommended skill mix

| Role | Skill | Use |
|------|--------|-----|
| Verify | `assets` | Confirm API/account reflects balance/permission-related state |
| Verify | `derivatives-trading-usds-futures` / `derivatives-trading-coin-futures` | Positions, order errors, futures capability checks |
| Verify | `sub-account` | Sub-account API vs sub-account structure |
| Environment | `healthcheck` | Local/runtime security baseline (not trading logic) |
| Extension | `skill-vetter` | Pre-install review of non-official skills |

---

## Plan

> Aligns with `Task_upgrade_advice.md` §2: classify error → permission truth → per-market read path → sub-account/IP → signing environment → supply-chain tools.

### Status checks and when you cannot proceed

- **Before planning**: Error code / HTTP status; which account owns the key (main/sub); whether “enable trading” is on; whether IP allowlist matches the caller environment.
- **If read path still fails** (permissions vs business mismatch): (1) State verified layers and failure layer; (2) Ask whether API rights were checked in the app, whether a sub-account key is needed, whether local time is synced; (3) Separate “exchange-side limits” vs “local code/signature”—do not claim “fixed” when unclear.
- **Cross-task rules**: See [Task_upgrade_advice.md](./Task_upgrade_advice.md) opening.

### A. Structured pipeline (DAG)

**Step 0: Prerequisite state check — *MANDATORY***

> **Goal**: Before concrete steps, understand the user’s full state for safer guidance.
> **Core skills**: `assets`, `spot` (for `getOrders`), `derivatives-trading-usds-futures` (for `getPositions`)

1. **Account assets**: `assets.getUserAssets` across wallets (especially `SPOT`, `FUNDING`).
2. **Open positions**: `derivatives-trading-usds-futures.getPositions` for USDS-M.
3. **Recent activity**: `spot.getOrders` for habits and open orders.

> **After diagnosis**: Adjust next steps; if underfunded, see **`fuzzy-intent-and-account-onboarding.md`**.

---

| Step | Action |
|------|--------|
| **Classify** | Map errors to: `-1022` signature, `-2015` key, timestamp, IP, sub-account key mismatch, business codes—do not conflate. |
| **Permission truth** | Read-only: `apiRestrictions`, `apiTradingStatus`, `account/status`. |
| **Read path** | Spot `GET /api/v3/account` → USDS-M `fapi/v2/account` or `positionRisk` → COIN-M `dapi/v1/account`; **failure layer = problem layer**. |
| **Sub-account** | `sub-account/list` + `ipRestriction` (GET) to align email ↔ key ↔ IP. |
| **Signature / env** | Time sync, `recvWindow`, secret; **P2P SAPI params are not sorted**. |
| **Supply chain** | `skill-vetter` / `healthcheck` vs exchange verification—**separate conclusions**. |

### B. Endpoint quick reference

1. **Verify API key permissions and trading switch (`assets`)**
   - `GET /sapi/v1/account/apiTradingStatus`: whether trading is allowed.
   - `GET /sapi/v1/account/apiRestrictions`: key permission scope.
   - `GET /sapi/v1/account/status`: account status.

2. **Verify spot readability (`spot`)**
   - `GET /api/v3/account`: if balances return, spot USER_DATA is readable.

3. **Verify USDS-M (`derivatives-trading-usds-futures`)**
   - `GET /fapi/v2/account` or `GET /fapi/v3/account`; `GET /fapi/v2/positionRisk`; on order failure compare `GET /fapi/v1/apiTradingStatus`.

4. **Verify COIN-M (`derivatives-trading-coin-futures`)**
   - `GET /dapi/v1/account`, `GET /dapi/v1/balance`.

5. **Sub-account + IP (`sub-account`)**
   - `GET /sapi/v1/sub-account/list`; `GET /sapi/v2/sub-account/subAccountApi/ipRestriction` with `email` + `subAccountApiKey`; POST/DELETE IP list per SKILL.

6. **`healthcheck` / `skill-vetter`**: per each SKILL (not expanded as REST here); complements HTTP error troubleshooting.

### C. `binance-cli` auth and environment (`binance` skill)

If the user uses **`binance-cli`** (`skills/binance/binance/SKILL.md`), auth and profiles follow **`references/auth.md`**:

- **Env**: `BINANCE_API_KEY`, `BINANCE_API_SECRET`; optional `BINANCE_API_ENV` (`prod` | `testnet` | `demo`).
- **Profile**: `binance-cli profile create`, `change`, `view`; `--profile <name>` on commands.
- **Security**: Do not log secrets; prefer read-only first (e.g. `binance-cli spot get-account`) before writes.

---

## Usage guide

### Structure

- **One failure layer at a time**: e.g. if `dapi` works but `fapi` does not → futures permission or URL, not spot signature first.
- **Troubleshoot without orders**: default read-only verification; test orders only as last resort.

### Errors and APIs

- **Error semantics**: e.g. `-2015` per official docs; `-1022` → HMAC, param order, `recvWindow`, time sync.
- **P2P SAPI signing**: for `/sapi/v1/c2c/...`, **do not sort** parameters (see `p2p` SKILL).
- **Read-first**: avoid `POST` orders in triage; use `spot` `POST /api/v3/order/test`, `/fapi/v1/order/test` when needed.
- **With `account-and-asset-management.md` / `trading-execution.md`**: after permissions OK, assets → former; trading → latter.
