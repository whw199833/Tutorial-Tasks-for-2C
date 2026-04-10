# Binance Square Posting

## Description

**Task summary**: Draft posts, multilingual posting, scheduling, news scrape-then-publish to Binance Square; **`square-post`** is the publish core; images, fetch, translation are general agent capabilities.

**Typical intents**: Write and publish Square articles; multilingual scheduled posts; news optimized for Square.

---

## Recommended skill mix

| Role | Skill | Use |
|------|--------|-----|
| Primary | `square-post` | Publish to Binance Square |
| Secondary | — | Images, translation, summary—agent, not mandatory skill |

**binance-skills-hub**: **`skills/binance/square-post/SKILL.md`**.

---

## Plan

> Aligns with `Task_upgrade_advice.en.md` §10: **draft finalize → self-check length/sensitive words → optional market facts → single `content/add`**; scheduled = external scheduler repeating flow.

### Status checks and when you cannot proceed

- **Before planning**: Account/Square permissions per SKILL/API; if citing prices, pull citeable facts per `market-data-and-analysis.en.md`.
- **If `content/add` fails** (sensitive word/length): (1) Error type; (2) User accepts edit/shorter text?; (3) **Do not** republish without confirmed draft.
- **Cross-task rules**: [Task_upgrade_advice.en.md](./Task_upgrade_advice.en.md).

### A. Structured pipeline (DAG)

**Step 0: Prerequisite state check — *MANDATORY*** when post ties to portfolio narrative.

---

| Step | Action |
|------|--------|
| **Draft** | Body, disclaimers (if market) in agent; check `square-post` error table. |
| **Data** | Cite prices/ranks: `market-data-and-analysis.en.md` first, embed in `bodyTextOnly`. |
| **Publish** | One `content/add`; success → URL from `data.id`. |
| **Schedule** | No official schedule API → cron/external fires `content/add` at time. |

### B. Endpoint quick reference

1. **Post**: `POST https://www.binance.com/bapi/composite/v1/public/pgc/openApi/content/add` — headers `X-Square-OpenAPI-Key`, `Content-Type: application/json`, `clienttype: binanceSkill`; body `bodyTextOnly` (plain text). Success `code`=`000000`; link `https://www.binance.com/square/post/{id}`.
2. **Errors**: `10005` KYC; `20002`/`20022` sensitive words; `20013` length—see SKILL table.
3. **Schedule / news scrape**: no Binance REST in this skill—external or manual.
4. **Market cites**: `market-data-and-analysis.en.md` then `content/add`.

### C. Scheduled posting (Python / Shell)

Default: user **cron** or **Python** at time T builds body and calls `content/add` (same as §A). Store `X_SQUARE_OPENAPI_KEY` in env; see shell/Python skeletons in Chinese doc.

---

## Usage guide

- **Content before API**: avoid unreviewed `content/add` hitting filters.
- **Do not invent numbers**: figures from `market-data-and-analysis.en.md` or public APIs.
- **Keys**: Square OpenAPI key ≠ trading `BINANCE_API_KEY`.
- **Single write path**: the one `content/add` endpoint above.
