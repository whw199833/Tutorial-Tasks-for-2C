# Binance Square Posting（内容创作与社交媒体）

## Description

**任务说明**：撰写帖子、多语言发帖、定时任务、新闻抓取后优化发布至 Binance Square 等意图；以 `square-post` 为发布能力核心，其余（配图建议、抓取、翻译）由 Agent 能力配合完成。

**典型线上意图**：撰写并发布币安广场文章；多语言定时发帖；新闻抓取优化后发布至 Square。

---

## 推荐 Skills 组合

| 角色 | Skill | 用途 |
|------|--------|------|
| 主路径 | `square-post` | 发布内容到 Binance Square |
| 辅助 | — | 配图、翻译、摘要为 Agent 通用能力，不强制 Skill |

**binance-skills-hub**：`square-post` → **`skills/binance/square-post/SKILL.md`**。

---

## Plan（执行计划）

> 与 `Task_upgrade_advice.cn.md` §10 对齐：**撰稿定稿 → 敏感词/长度自检 → 如需则先拉行情事实 → 单点 `content/add`**；定时发帖 = 外部调度重复同一流程。

### 状态确认与不支持时的沟通

- **规划前需确认**：发帖是否需账号/Square 侧权限（以 Skill 与接口返回为准）；引用行情时是否已按 `market-data-and-analysis.cn.md` 拉取可引用事实。
- **若 `content/add` 失败（敏感词/长度等）**：① 说明错误类型；② 追问用户是否接受修改稿或缩短正文；③ **不在**未确认定稿前重复调用发布。
- **跨 Task 原则**：见 [Task_upgrade_advice.cn.md](./Task_upgrade_advice.cn.md) 开篇。

### A. 结构化流水线（DAG）

**Step 0: 前置状态诊断 (Prerequisite State Check) - *MANDATORY***
> **目标**: 在执行任何具体任务前，必须先全面了解用户的当前状态，以提供个性化、避免风险的建议。
> **核心 Skills**: `assets`, `spot` (for `getOrders`), `derivatives-trading-usds-futures` (for `getPositions`)

1.  **账户资产查询**: 调用 `assets.getUserAssets`，检查各钱包（特别是现货 `SPOT` 和资金 `FUNDING`）的可用余额、总估值。
2.  **当前持仓分析**: 调用 `derivatives-trading-usds-futures.getPositions`，检查用户是否有U本位合约持仓，了解其方向、大小和未实现盈亏。
3.  **历史交易与挂单**: 调用 `spot.getOrders`，检查用户近期的交易习惯（如偏好的币对）和当前有无未成交的挂单。

> **诊断后决策**: 根据诊断结果动态调整后续步骤。例如，如果用户已有相关持仓，应优先围绕该持仓展开计划；如果资金不足，则参考 `fuzzy-intent-and-account-onboarding.cn.md` 进行入金引导。

---


| 步骤 | 动作 |
|------|------|
| **撰稿** | 正文、免责声明（若涉行情）在 Agent 侧完成；对照 `square-post` 错误码（敏感词、长度）。 |
| **数据** | 引用价/榜：先 `market-data-and-analysis.cn.md` 拉取可引用事实，**写入** `bodyTextOnly`。 |
| **发布** | `content/add` 一次；成功用 `data.id` 生成 URL。 |
| **定时** | 无官方定时 API → cron/外部调度在设定时刻再调 `content/add`。 |

### B. 接口级速查

1. **发帖（`square-post`）**
   - `POST https://www.binance.com/bapi/composite/v1/public/pgc/openApi/content/add`
   - Headers：`X-Square-OpenAPI-Key`、`Content-Type: application/json`、`clienttype: binanceSkill`
   - Body JSON：`bodyTextOnly`（必填，纯文本正文）。
   - 成功：`code`=`000000`，用 `data.id` 拼帖子链接 `https://www.binance.com/square/post/{id}`。

2. **错误码**：`10005` 需 KYC；`20002`/`20022` 敏感词；`20013` 长度限制（见 SKILL 表）。

3. **定时发布 / 抓取新闻**：无 Binance 公开 REST 在本 Skill 中；用外部调度或手动。

4. **引用行情**：先 `market-data-and-analysis.cn.md`（如 `unified/rank/list` 或 `token/search`），再写作并调用上述 `content/add`。

### C. 定时发帖的默认实现（Python / Shell）

无官方「预约发帖」接口时，**默认**用用户环境的 **Shell + cron** 或 **Python** 在指定时刻构造正文并调用 `content/add`（与 §A「定时 = 外部调度」一致）。

| 方式 | 说明 |
|------|------|
| **Shell** | `crontab` 在整点/每天某时刻执行 `post_square.sh`：从文件或 `heredoc` 读入 `bodyTextOnly`，用 `curl` 带 `X-Square-OpenAPI-Key` 发 JSON。 |
| **Python** | 同一时刻由 `schedule`/`APScheduler` 或 **cron 调 `python post.py`**：脚本内 `requests.post` 到发帖 URL，正文可来自模板文件或先生成再提交。 |

密钥使用环境变量（如 `SQUARE_OPENAPI_KEY`），勿写入仓库。

**Shell 示例骨架**：

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

**Python 示例骨架**：

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

## 使用指南

### 结构化要点

- **先内容后接口**：避免未审稿就调用 `content/add` 触发敏感词/长度错误。
- **行情可引用不可编造**：数字必须来自 `market-data-and-analysis.cn.md` 或公开行情接口。

### 密钥与接口

- **密钥**：Square OpenAPI Key 与交易 API Key 不同；勿混用 `BINANCE_API_KEY`。
- **单一写接口**：发帖只依赖上述一条 `content/add`。
