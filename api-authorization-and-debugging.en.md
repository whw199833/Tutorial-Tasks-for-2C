# API Authorization and Debugging

## Description

**Task**: “Connection and permission” issues around API keys, trading permissions, signature failures, sub-account APIs, and futures order errors—help determine whether the issue is account permissions, key configuration, or client-side calls; within automation limits, pair with balance and position reads.

**Typical intents**: authorize AI trading with profit targets; API key setup; API config and call failures; sub-account BNB futures and API issues; account authorization and signature debugging; trade execution failures and signatures; query positions and debug API authorization.

**Skill boundaries**: No single Skill “generates or fixes” user local code; `healthcheck` is environment hardening; `assets` / derivatives Skills **verify** that authorization works (balances, positions, error semantics). `skill-vetter` reviews third-party Skill install risk, distinct from exchange API debugging.

---

## Recommended Skills

| Role | Skill | Use |
|------|--------|-----|
| Verify | `assets` | Whether API/account reflects balance/permission-related state |
| Verify | `derivatives-trading-usds-futures` / `derivatives-trading-coin-futures` | Positions, order errors, futures capabilities |
| Verify | `sub-account` | Whether sub-account API matches structure |
| Environment | `healthcheck` | Local/runtime safety checks (not trading logic) |
| Extension | `skill-vetter` | Review non-official Skills before install |

---

## Plan

> Aligned with `Task_upgrade_advice.en.md` §2: classify errors → permission ground truth → read path per market → sub-account/IP → signing environment → supply-chain tools.

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Classify** | Map errors to: `-1022` signature, `-2015` key, timestamp, IP, sub-account key mismatch, business codes; do not mix. |
| **Ground truth** | Read-only: `apiRestrictions`, `apiTradingStatus`, `account/status`. |
| **Read path** | Spot `GET /api/v3/account` → USDS-M `fapi/v2/account` or `positionRisk` → COIN-M `dapi/v1/account`; **the failure layer is the problem layer**. |
| **Sub-account** | `sub-account/list` + `ipRestriction` (GET) to verify email ↔ key ↔ IP. |
| **Signing / env** | Time sync, `recvWindow`, Secret; **do not sort P2P SAPI params**. |
| **Supply chain** | `skill-vetter` / `healthcheck` vs exchange verification — **separate conclusions**. |

### B. API quick reference

1. **Verify API key permissions and trading switch (`assets`)**
   - `GET /sapi/v1/account/apiTradingStatus`: whether trading is allowed.
   - `GET /sapi/v1/account/apiRestrictions`: key permission scope.
   - `GET /sapi/v1/account/status`: account status.

2. **Verify spot readability (`spot`)**
   - `GET /api/v3/account`: if balances return, spot USER_DATA is readable.

3. **Verify USDS-M (`derivatives-trading-usds-futures`)**
   - `GET /fapi/v2/account` or `GET /fapi/v3/account`: futures account entry.
   - `GET /fapi/v2/positionRisk`: positions; on order failure compare `GET /fapi/v1/apiTradingStatus`.

4. **Verify COIN-M (`derivatives-trading-coin-futures`)**
   - `GET /dapi/v1/account`, `GET /dapi/v1/balance`: COIN-M account and balances.

5. **Sub-account + IP allowlist (`sub-account`)**
   - `GET /sapi/v1/sub-account/list`: confirm sub-account emails.
   - `GET /sapi/v2/sub-account/subAccountApi/ipRestriction`: `email` + `subAccountApiKey` for IP limits.
   - `POST` same family: add IP; `DELETE .../ipList`: remove IP.

6. **`healthcheck` / `skill-vetter`**: follow each SKILL (not expanded as REST here); complements exchange HTTP error debugging.

---

## Usage

### Structured notes

- **One failure layer at a time**: e.g. `dapi` OK but `fapi` fails → futures permission or URL, do not change spot signing first.
- **Debug, don’t trade**: default read-only verification; test orders only as last resort.

### APIs & error codes

- **Semantics**: map `-2015` etc. to Binance official docs; `-1022` → HMAC, param order, `recvWindow`, local time sync.
- **P2P SAPI signing**: for `p2p` `/sapi/v1/c2c/...`, **do not sort parameters** (see `p2p` SKILL), unlike standard Spot/Futures.
- **Read-first**: avoid `POST` orders in debug; test endpoints if needed: `spot` `POST /api/v3/order/test`, `/fapi/v1/order/test` (pick market Skill).
- **With `account-and-asset-management.en.md` / `trading-execution.en.md`**: after permissions OK, assets → former; trading → latter.
