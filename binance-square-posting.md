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

---

## Plan（执行计划）

> 与 `Task_upgrade_advice.md` §10 对齐：**撰稿定稿 → 敏感词/长度自检 → 如需则先拉行情事实 → 单点 `content/add`**；定时发帖 = 外部调度重复同一流程。

### A. 结构化流水线（DAG）

| 步骤 | 动作 |
|------|------|
| **撰稿** | 正文、免责声明（若涉行情）在 Agent 侧完成；对照 `square-post` 错误码（敏感词、长度）。 |
| **数据** | 引用价/榜：先 `market-data-and-analysis.md` 拉取可引用事实，**写入** `bodyTextOnly`。 |
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

4. **引用行情**：先 `market-data-and-analysis.md`（如 `unified/rank/list` 或 `token/search`），再写作并调用上述 `content/add`。

---

## 使用指南

### 结构化要点

- **先内容后接口**：避免未审稿就调用 `content/add` 触发敏感词/长度错误。
- **行情可引用不可编造**：数字必须来自 `market-data-and-analysis.md` 或公开行情接口。

### 密钥与接口

- **密钥**：Square OpenAPI Key 与交易 API Key 不同；勿混用 `BINANCE_API_KEY`。
- **单一写接口**：发帖只依赖上述一条 `content/add`。
