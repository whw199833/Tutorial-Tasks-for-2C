# Education and Learning

## Description

**Task**: Study plans, reading lists, term definitions, investing basics, language learning—**explanation and structured output**; optional quote Skills as teaching examples, not mandatory.

**Typical intents**: stock/research study plans and materials; trading terms; language learning and expression analysis.

---

## Recommended Skills

| Scenario | Skill | Use |
|----------|--------|-----|
| Example demo | `query-token-info` | Real quotes for K-line, volume concepts |
| Example demo | `crypto-market-rank` | Ranks, heat, volatility |
| Extension | `skill-creator` | User wants knowledge as reusable Agent Skill |
| General | — | Terms and general knowledge need no Skill |

---

## Plan

> Aligned with `Task_upgrade_advice.en.md` §11: **concepts first** → on request: **search → dynamic → K-line** chain; rank teaching: single `unified/rank`; reusable tutorials: `skill-creator`.

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Default** | Terms, study plans, outlines → **no API**. |
| **Demo chain** | Pick a major: `search` → `dynamic` explain fields → **then** `K-line` OHLCV. |
| **Rank teaching** | One `unified/rank`, explain `rankType` / `period` / `sortBy`. |
| **Boundary** | State **teaching is not buy/sell advice**; `skill-creator` only when user wants reusable tutorials. |

### B. API quick reference

1. **Terms & general**: zero API; explain directly.

2. **Optional example: single token**
   - `GET https://web3.binance.com/bapi/defi/v5/public/wallet-direct/buw/wallet/market/token/search/ai?keyword=BTC`
   - `GET .../bapi/defi/v4/public/wallet-direct/buw/wallet/market/token/dynamic/info/ai?chainId=&contractAddress=` (volume, % change, holders).

3. **Optional example: rank**
   - `POST .../unified/rank/list/ai`, show how `rankType`, `period` affect sort.

4. **Optional example: K-line**
   - `GET https://dquery.sintral.io/u-kline/v1/k-line/candles` (params per `query-token-info` SKILL: OHLCV index meaning).

5. **Solidify as Skill**: Cursor `skill-creator` (not Binance HTTP).

---

## Usage

### Structured notes

- **API only when asked for examples**: reduce noise if user did not ask for live data.
- **Fixed demo order**: explain dynamic fields before K-line, do not skip straight to chart.

### Compliance

- **Neutral**: do not interpret API output as “must rise / must buy.”
