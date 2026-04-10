# On-chain Signals and Security

## Description

**Task summary**: BSC-style transfer patterns, smart-money signals, token security audit, address holdings; and **data/signal parts** of monitoring/alert intents (long-running scripts still depend on user env and cron/serverless).

**Typical intents**: BSC transfer lookup; smart-money monitoring/alerts; scheduled monitoring/push; buy-signal notifications; token audit; smart-money + token analysis.

---

## Recommended skill mix

| Role | Skill | Use |
|------|--------|-----|
| Primary | `trading-signal` | Smart-money buy/sell signals |
| Primary | `query-token-audit` | Honeypot/rug-style risk |
| Primary | `query-address-info` | On-chain address assets/holdings |
| Secondary | `query-token-info` | Token context and price |
| Secondary | `meme-rush` | Meme lane heat |

**binance-skills-hub**: all under **`skills/binance-web3/<skill>/SKILL.md`**; exchange order placement not in this task.

---

## Plan

> Aligns with `Task_upgrade_advice.en.md` §5: **contract → audit first**; then signals; `dynamic` liquidity on signal tokens; address paging; meme may align narrative.

### Status checks and when you cannot proceed

- **Before planning**: If **contract address**, **audit** first; if user will **trade on exchange**, also confirm account state (`trading-execution.en.md`).
- **If audit high-risk or liquidity very low**: (1) State risk; (2) Ask if they still want signal/monitor view; (3) **Do not** imply safe to size up; ask for addresses to monitor if missing.
- **Cross-task rules**: [Task_upgrade_advice.en.md](./Task_upgrade_advice.en.md).

### A. Structured pipeline (DAG)

**Step 0: Prerequisite state check — *MANDATORY*** — `assets`, `spot`, `derivatives-trading-usds-futures` when tying to account/execution; else optional for pure on-chain read.

---

| Step | Action |
|------|--------|
| **Audit first** | If `contractAddress`: **`query-token-audit` first**; honeypot/high risk → **risk first**, then decide signals. |
| **Smart money** | `trading-signal` → sort/filter by `direction`, `maxGain`, `exitRate`, `alertPrice` vs `currentPrice`. |
| **Deepen** | For picks: `dynamic/info` liquidity; very low → soften “actionable” wording. |
| **Address** | `query-address-info` paging; continuous monitor = user **poll** same API (cron)—skill does not host daemons. |
| **Meme** | New launch talk: `meme-rush` stage vs signal list same narrative? |

### B. Endpoint quick reference

1. **`query-token-audit`**: `POST https://web3.binance.com/bapi/defi/v1/public/wallet-direct/security/token/audit` — `binanceChainId`, `contractAddress`, UUID `requestId`.
2. **`trading-signal`**: `POST .../smart-money/ai` — `chainId`, `page`, `pageSize`.
3. **`query-address-info`**: `GET .../address/pnl/active-position-list/ai` — `address`, `chainId`, `offset`.
4. **`query-token-info`**: search/dynamic per `market-data-and-analysis.en.md` for signal token context.
5. **`meme-rush`**: rank endpoints per SKILL.
6. **Monitoring**: all **pull** REST; scheduled push = user cron/serverless polling—no WebSocket hosting in skill.

### C. Default scheduling (Python / Shell)

“Scheduled monitor / conditional alert / timed push” defaults to **user-owned** scripts; agent/skill does **single** pulls, **no** hosted service.

| Method | Use | Notes |
|--------|-----|--------|
| **Shell + cron** | Fixed interval | `crontab` → `.sh` → `curl` + `jq`; webhook/log per user. |
| **Python** | Poll/threshold | `requests`/`httpx` loop or APScheduler; alerts (DingTalk/TG/mail) user-defined. |

Do **not** hardcode API keys; use env/secrets. Example skeletons in Chinese doc apply with same URLs.

---

## Usage guide

- **Audit ≠ tradable**: pass is technical scan only, not returns or endorsement.
- **Signals + liquidity**: do not over-read smart-money direction on illiquid tokens.
- **Chain IDs**: BSC `56`, Base `8453`, Solana `CT_501`, Ethereum `1` (audit).
- **With `market-data-and-analysis.en.md`**: unified/social for breadth; this task for address/signal depth.
- **Privacy**: redact wallet addresses/holdings when showing users if needed.
