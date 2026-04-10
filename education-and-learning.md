# Education and Learning

## Description

**Task summary**: Study plans, reading lists, term definitions, investing literacy, language learning—**explain and structure** first; optional market skills as **teaching examples**, not mandatory.

**Typical intents**: Equity/research study plans; trading glossary; language learning and expression analysis.

---

## Recommended skill mix

| Scenario | Skill | Use |
|----------|--------|-----|
| Demo | `query-token-info` | Real klines/volume to explain concepts |
| Demo | `crypto-market-rank` | Rankings, heat, volatility |
| Extension | `skill-creator` | User wants reusable agent skill from material |
| General | — | Terms and general knowledge—no skill |

**binance-skills-hub**: demos in **`skills/binance-web3/query-token-info`** etc.; `skill-creator` **not** in hub (see README).

---

## Plan

> Aligns with `Task_upgrade_advice.md` §11: **concepts first** → if user wants examples: **search → dynamic → klines**; ranking lesson one `unified/rank`; reusable tutorial → `skill-creator`.

### Status checks and when you cannot proceed

- **Before planning**: Pure concepts/terms/plans → usually no account; if “size for my money” or “how much to buy”, confirm balance and goals (`trading-execution.md`).
- **If info thin**: Ask example vs real account; until clear, **only** formulas and conditional examples.
- **Cross-task rules**: [Task_upgrade_advice.md](./Task_upgrade_advice.md).

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Default** | Terms, study plan, structure → **no API**. |
| **Demo chain** | Pick mainstream: `search` → `dynamic` explain fields → **then** kline OHLCV. |
| **Ranking lesson** | One `unified/rank`—explain `rankType` / `period` / `sortBy`. |
| **Boundary** | State **teaching ≠ buy/sell advice**; `skill-creator` for reusable tutorials. |

### B. Endpoint quick reference

1. **Terms**: zero API.
2. **Optional single token**: `search/ai`, `dynamic/info` per `query-token-info` SKILL.
3. **Optional rank**: `POST .../unified/rank/list/ai`.
4. **Optional klines**: `GET https://dquery.sintral.io/u-kline/v1/k-line/candles` per SKILL.
5. **Skill packaging**: Cursor `skill-creator`—not Binance HTTP.

---

## Usage guide

- **API only when asked for examples**—reduce cognitive load otherwise.
- **Demo order**: explain fields before klines.
- **Neutral**: API fields are not “must go up / must buy”.
