# Token Research and Opportunities

## Description

**Task**: Alts and sectors (AI, DeFi, Meme), new launches and Alpha, risk/reward and entry discussion—**research** intents; Skills supply **data, signals, audit, ranks**; Agent supplies framework and risk wording, not deterministic investment advice.

**Typical intents**: alt investment ideas; new tokens and trends; AI tokens; DeFi inflows; Meme and new launches; PRL-style targets and entries; risk/reward and sizing judgments.

---

## Recommended Skills

| Role | Skill | Use |
|------|--------|-----|
| Primary | `query-token-info` | Price, volume, K-lines |
| Primary | `crypto-market-rank` | Sector heat and relative strength |
| Primary | `trading-signal` | Smart-money side picture |
| Primary | `query-token-audit` | Contract and rug risk |
| Context | `meme-rush` | Meme and new launches |
| Context | `alpha` | Binance Alpha token context |
| Context | `binance-tokenized-securities-info` | RWA / tokenized equity research |

---

## Plan

> Aligned with `Task_upgrade_advice.en.md` §6: **locate → risk (audit) → market position → behavior/narrative**; RWA isolated; close as **facts + hypotheses**; execution → `trading-execution.en.md`.

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Locate** | `search/ai` or keyword → `chainId`+`contractAddress`; Alpha context: parallel `alpha` `token/list` + `ticker`/`klines`. |
| **Risk** | `query-token-audit`: **fail** → answer risk-heavy, light on “opportunity.” |
| **Market position** | `unified/rank` (keyword/sector filter) + `inflow/rank` (`tagType=2`) → percentile, relative strength. |
| **Behavior / narrative** | `trading-signal`; Meme: `meme-rush` or exclusive rank; social: optional `social/hype`. |
| **RWA** | Only `binance-tokenized-securities-info` list→dynamic→K-line, **do not** mix with Meme copy. |
| **Close** | **Fact layer (APIs) + hypothesis layer (risk, sizing bands)**; orders → `trading-execution.en.md`. |

### B. API quick reference

1. **Locate**
   - `query-token-info`: `GET .../token/search/ai?keyword=`; confirm `chainId` + `contractAddress`.
   - `alpha` (Alpha context only): `GET https://www.binance.com/bapi/defi/v1/public/alpha-trade/get-exchange-info`; `GET https://www.binance.com/bapi/defi/v1/public/alpha-trade/ticker?symbol=`; `GET https://www.binance.com/bapi/defi/v1/public/alpha-trade/klines`; `GET https://www.binance.com/bapi/defi/v1/public/wallet-direct/buw/wallet/cex/alpha/all/token/list` (`symbol` like `ALPHA_175USDT` from token list).

2. **Quotes & sectors**
   - `crypto-market-rank`: `POST .../unified/rank/list/ai` (`rankType` 10/11/20/40, `keywords`/`excludes`).
   - Smart-money inflow: `POST .../tracker/wallet/token/inflow/rank/query/ai` (`tagType: 2`).

3. **Signals & risk**
   - `query-token-audit`: `POST .../security/token/audit` (UUID `requestId` required).
   - `trading-signal`: `POST .../smart-money/ai`.
   - Meme: `POST .../pulse/rank/list/ai` (meme-rush); or `GET .../exclusive/rank/list/ai`.

4. **RWA**
   - `binance-tokenized-securities-info`: `GET https://www.binance.com/.../rwa/stock/detail/list/ai` → dynamic/K per SKILL (API 1→5/6).

5. **Orders**: `trading-execution.en.md` (`spot`/`fapi`/`dapi`/`eapi`); this Task does not execute trade REST.

---

## Usage

### Structured notes

- **API ≠ instruction**: do not turn response fields into “must buy/sell.”
- **Audit conflict**: if `auditInfo` disagrees with dedicated `query-token-audit`, **prefer dedicated audit**.

### Fields

- **Research output**: cite returned fields (`riskLevelEnum`, `inflow`, `percentChange24h`); do not invent metrics.
- **Alpha**: use `bapi/defi/v1/public/alpha-trade/*` and CEX Alpha list; do not confuse with Web3 `rankType=20` “Alpha token” leaderboard dimension.
