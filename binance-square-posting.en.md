# Binance Square Posting

## Description

**Task**: Write posts, multi-language posting, scheduled tasks, news scrape then optimize for Binance Square; **`square-post`** is the core publish capability; images, scrape, translation use Agent capabilities.

**Typical intents**: write and publish Square articles; multi-language scheduled posts; news optimized then published to Square.

---

## Recommended Skills

| Role | Skill | Use |
|------|--------|-----|
| Primary | `square-post` | Publish to Binance Square |
| Secondary | — | Images, translation, summary = general Agent, not mandatory Skill |

---

## Plan

> Aligned with `Task_upgrade_advice.en.md` §10: **draft final** → sensitive-word/length self-check → optional market facts → single `content/add`; scheduled = external scheduler repeating the same flow.

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Draft** | Body, disclaimers (if market-related) in Agent; check `square-post` errors (sensitive words, length). |
| **Data** | For price/rank: pull facts from `market-data-and-analysis.en.md`, **embed** in `bodyTextOnly`. |
| **Publish** | One `content/add`; success → use `data.id` for URL. |
| **Schedule** | No official schedule API → cron / external scheduler calls `content/add` at time. |

### B. API quick reference

1. **Post (`square-post`)**
   - `POST https://www.binance.com/bapi/composite/v1/public/pgc/openApi/content/add`
   - Headers: `X-Square-OpenAPI-Key`, `Content-Type: application/json`, `clienttype: binanceSkill`
   - Body JSON: `bodyTextOnly` (required, plain text).
   - Success: `code`=`000000`, build post URL `https://www.binance.com/square/post/{id}` from `data.id`.

2. **Errors**: `10005` KYC; `20002`/`20022` sensitive words; `20013` length (see SKILL table).

3. **Schedule / news scrape**: no Binance REST in this Skill; external scheduler or manual.

4. **Quotes**: first `market-data-and-analysis.en.md` (e.g. `unified/rank/list` or `token/search`), then write and `content/add`.

### C. Scheduled posting defaults (Python / Shell)

There is no official “schedule post” API. **By default**, use **Shell + cron** or **Python** on the user’s side to build `bodyTextOnly` at a chosen time and call `content/add` (same as §A: schedule = external orchestration).

| Approach | Notes |
|----------|--------|
| **Shell** | `crontab` runs `post_square.sh` at a fixed time: read body from a file or heredoc, `curl` with `X-Square-OpenAPI-Key` and JSON body. |
| **Python** | Same moment via `schedule` / **APScheduler** or **cron** calling `python post.py`: `requests.post` to the content endpoint; body from a template file or generated first. |

Load keys from the environment (e.g. `X_SQUARE_OPENAPI_KEY`); do not commit secrets.

**Shell sketch**:

```bash
#!/usr/bin/env bash
set -euo pipefail
KEY="${X_SQUARE_OPENAPI_KEY:?set key in env}"
BODY_FILE="${1:?path to body text file}"
curl -sS -X POST "https://www.binance.com/bapi/composite/v1/public/pgc/openApi/content/add" \
  -H "X-Square-OpenAPI-Key: ${KEY}" \
  -H "Content-Type: application/json" \
  -H "clienttype: binanceSkill" \
  -d "$(jq -n --rawfile t "$BODY_FILE" '{bodyTextOnly: $t}')"
```

**Python sketch**:

```python
import os, pathlib, requests

def post_square(body: str) -> dict:
    url = "https://www.binance.com/bapi/composite/v1/public/pgc/openApi/content/add"
    headers = {
        "X-Square-OpenAPI-Key": os.environ["X_SQUARE_OPENAPI_KEY"],
        "Content-Type": "application/json",
        "clienttype": "binanceSkill",
    }
    r = requests.post(url, json={"bodyTextOnly": body}, headers=headers, timeout=60)
    r.raise_for_status()
    return r.json()

if __name__ == "__main__":
    text = pathlib.Path(os.environ.get("BODY_FILE", "post.txt")).read_text(encoding="utf-8")
    print(post_square(text))
```

---

## Usage

### Structured notes

- **Content before API**: avoid calling `content/add` before review to reduce sensitive-word/length errors.
- **Quotes must be real**: numbers must come from `market-data-and-analysis.en.md` or public quote APIs, not invented.

### Keys

- **Keys**: Square OpenAPI Key ≠ trading API Key; do not reuse `BINANCE_API_KEY`.
- **Single write**: posting uses the one `content/add` endpoint above.
