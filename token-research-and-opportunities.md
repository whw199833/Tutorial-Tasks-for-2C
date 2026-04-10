# Token Research and Opportunities

## Description

**Task summary**: Alts and sectors (AI, DeFi, Meme), new launches and Alpha, risk/reward and entry talk—**research** intents; skills supply **data, signals, audit, rankings**; agent supplies framework and risk wording, **not** definitive investment advice.

**Typical intents**: Alt “investment ideas”; new tokens and trends; AI-theme tokens; DeFi inflows; Meme/new launches; specific tickers, entries, position sizing, etc.

---

## Recommended skill mix

| Role | Skill | Use |
|------|--------|-----|
| Primary | `query-token-info` | Price, volume, klines |
| Primary | `crypto-market-rank` | Sector heat, relative strength |
| Primary | `trading-signal` | Smart-money side |
| Primary | `query-token-audit` | Contract/rug risk |
| Context | `meme-rush` | Meme/new launch |
| Context | `alpha` | Binance Alpha tokens |
| Context | `binance-tokenized-securities-info` | RWA / tokenized equities |

**binance-skills-hub**: web3 skills under **`skills/binance-web3/`**; `alpha` → **`skills/binance/alpha/`**; execution → **`trading-execution.md`** (incl. **`binance` CLI**).

---

## Plan

> Aligns with `Task_upgrade_advice.md` §6: **locate → audit → market position → behavior/narrative**; RWA separate; close with **facts + assumptions**; orders → `trading-execution.md`.

### Status checks and when you cannot proceed

- **Before planning**: Separate research from “place orders”; if user wants **executable plan** (size, leverage), confirm **balance, orders, risk** (`trading-execution.md`).
- **If audit fails or data thin**: (1) Risk disclosure first; (2) Ask if they still want narrative/price layer; (3) **No** definitive buy plan without funding state.
- **Cross-task rules**: [Task_upgrade_advice.md](./Task_upgrade_advice.md).

### A. Structured pipeline (DAG)

**Step 0: Prerequisite state check — *MANDATORY*** when user ties research to account or execution.

---

| Step | Action |
|------|--------|
| **Locate** | `search/ai` or keywords → `chainId`+`contractAddress`; Alpha: `alpha` `token/list` + `ticker`/`klines`. |
| **Risk** | `query-token-audit`; **fail** → risk-led answer, less “opportunity”. |
| **Market position** | `unified/rank` + `inflow/rank` (`tagType=2`) → percentile, relative strength. |
| **Behavior/narrative** | `trading-signal`; Meme: `meme-rush` or exclusive rank; optional `social/hype`. |
| **RWA** | Only `binance-tokenized-securities-info` pipeline—**do not** mix with Meme copy. |
| **Close** | **Facts (API) + assumptions (risk, sizing bands)**; execution → `trading-execution.md`. |

### B. Endpoint quick reference

1. **Locate**: `query-token-info` `search/ai`; Alpha: `alpha-trade` exchange-info, ticker, klines, CEX alpha token list per SKILL.
2. **Market/sector**: `crypto-market-rank` `unified/rank/list/ai`; inflow `inflow/rank/query/ai` (`tagType: 2`).
3. **Signals/risk**: `query-token-audit`; `trading-signal`; Meme ranks per SKILL.
4. **RWA**: `binance-tokenized-securities-info` workflow (API 1→6 per SKILL).
5. **Orders**: `trading-execution.md` only—no trade REST in this task.

---

## Usage guide

- **API fields are not orders**: do not turn JSON into “must buy/sell”.
- **Audit conflicts**: prefer dedicated `query-token-audit` over generic `auditInfo`.
- **Research output**: cite fields (`riskLevelEnum`, `inflow`, `percentChange24h`); do not invent metrics.
- **Alpha**: use `bapi/defi/v1/public/alpha-trade/*` and CEX Alpha list—do not confuse with Web3 `rankType=20` “Alpha” leaderboard dimension.
