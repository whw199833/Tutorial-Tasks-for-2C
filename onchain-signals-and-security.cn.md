# On-chain Signals and Security（链上数据与智能资金监控）

## Description

**任务说明**：BSC 等链上转账思路、聪明钱/智能资金信号查询、代币安全审计、地址持仓查看；以及「监控脚本、定时推送、条件警报」类意图中**可由 Skills 提供的数据与信号部分**（实际长期脚本部署仍依赖用户环境与 cron/云函数）。

**典型线上意图**：BSC 转账查询方案；智能资金监控与警报；定时监控与推送；买入信号通知；代币安全审计；聪明钱信号与代币分析。

---

## 推荐 Skills 组合

| 角色 | Skill | 用途 |
|------|--------|------|
| 主路径 | `trading-signal` | 聪明钱买卖信号 |
| 主路径 | `query-token-audit` | 蜜罐、跑路风险等安全审计 |
| 主路径 | `query-address-info` | 链上地址资产与持仓 |
| 辅助 | `query-token-info` | 代币基础信息与价格上下文 |
| 辅助 | `meme-rush` | Meme 赛道热度与追踪 |

---

## Plan（执行计划）

> 与 `Task_upgrade_advice.cn.md` §5 对齐：**有合约则先审计**；再谈信号；信号代币用 `dynamic` 看流动性；地址监控走分页；Meme 可与信号对齐叙事。

### A. 结构化流水线（DAG）

| 步骤 | 动作 |
|------|------|
| **审计优先** | 若有 `contractAddress`：**先** `query-token-audit`；高风险/蜜罐则**先输出风险**，再决定是否展开信号。 |
| **聪明钱** | `trading-signal` 拉取 → 按 `direction`、`maxGain`、`exitRate`、`alertPrice` vs `currentPrice` **排序/过滤**。 |
| **加深** | 对入选合约：`dynamic/info` 看流动性；过低则降低「可跟进」表述强度。 |
| **地址** | `query-address-info` 分页；持续监控 = 用户侧 **轮询** 同一接口（cron），Skill 不托管守护进程。 |
| **Meme** | 用户谈新盘时：`meme-rush` 阶段与信号列表对齐是否同一叙事。 |

### B. 接口级速查

1. **代币安全审计（`query-token-audit`）**（有合约时优先调用）
   - `POST https://web3.binance.com/bapi/defi/v1/public/wallet-direct/security/token/audit`
   - JSON：`binanceChainId`（`56`/`8453`/`CT_501`/`1`）、`contractAddress`、`requestId`（UUID v4）；Header 可加 `source: agent`。

2. **聪明钱信号（`trading-signal`）**
   - `POST https://web3.binance.com/bapi/defi/v1/public/wallet-direct/buw/wallet/web/signal/smart-money/ai`
   - Body：`chainId`（`56` BSC、`CT_501` Solana）、`page`、`pageSize`；响应含 `alertPrice`、`currentPrice`、`maxGain`、`exitRate`、`direction` 等。

3. **地址持仓（`query-address-info`）**
   - `GET https://web3.binance.com/bapi/defi/v3/public/wallet-direct/buw/wallet/address/pnl/active-position-list/ai`
   - Query：`address`、`chainId`、`offset`；Headers：`clienttype: web`、`clientversion: 1.2.0`。

4. **补充行情（`query-token-info`）**
   - `GET .../market/token/search/ai` 或 `.../token/dynamic/info/ai`（见 `market-data-and-analysis.cn.md`）用于信号代币的现价与流动性。

5. **Meme 赛道（`meme-rush`）**
   - `POST .../buw/wallet/market/token/pulse/rank/list/ai`（`rankType` 10/20/30）；Topic 用 `GET .../social-rush/rank/list/ai`。

6. **监控/告警**：上述均为**拉取式 REST**；定时推送需用户侧 cron/云函数轮询同一接口，Skill 不包含 WebSocket 部署。

### C. 定时任务与监控的默认实现（Python / Shell）

本 Task 中的「定时监控、条件告警、定时推送」**默认**在用户自有环境（本机、VPS、云函数）用脚本实现；Agent/Skill 只负责**单次**拉数，**不**托管常驻服务。

| 方式 | 适用 | 说明 |
|------|------|------|
| **Shell + cron** | 固定周期拉取（如每 5 分钟） | `crontab` 调用 `.sh`，脚本内用 `curl` 请求 §B 中的 `POST`/`GET`，配合 `jq` 解析后决定是否 `echo`、写日志或调用用户自配 webhook。 |
| **Python** | 轮询、阈值判断、多接口编排 | 推荐 `requests` 或 `httpx` 循环请求同一 REST；或用 **APScheduler** / 系统 **cron** 单次执行 `python monitor.py`。告警出口（钉钉/TG/邮件/文件）由脚本自行对接，不在 Skill 范围内。 |

**约定**：密钥、频率与合规（请求间隔、勿刷屏）由部署方配置；脚本内**不要**硬编码 API Key，使用环境变量或密钥管理。

**Shell 示例骨架**（周期由 crontab 控制，脚本内只负责一次拉取）：

```bash
#!/usr/bin/env bash
set -euo pipefail
# 示例：拉聪明钱信号后按条件告警（URL/Body 以 query-token-info / trading-signal 的 SKILL 为准）
# export WEB3_COOKIE_OR_HEADERS=...  # 若接口需要，从环境注入
curl -sS -X POST "${SIGNAL_URL}" \
  -H "Content-Type: application/json" \
  -d "{\"chainId\":\"56\",\"page\":1,\"pageSize\":20}" | jq .
```

**Python 示例骨架**（长轮询可用 `while True` + `sleep`，生产环境更推荐 cron 调单次脚本）：

```python
import os, time, requests

def poll_once():
    url = os.environ["SMART_MONEY_URL"]  # 与 §B trading-signal 一致
    r = requests.post(url, json={"chainId": "56", "page": 1, "pageSize": 20}, timeout=30)
    r.raise_for_status()
    data = r.json()
    # TODO: 解析 direction / alertPrice vs currentPrice，命中规则则 notify
    return data

if __name__ == "__main__":
    while True:
        poll_once()
        time.sleep(300)  # 5 分钟；或用 cron 去掉循环，每次只执行一次
```

---

## 使用指南

### 结构化要点

- **审计 ≠ 可交易**：审计通过仍仅表示技术扫描结果，不代表收益或官方背书。
- **信号与流动性一起看**：避免在极低流动性标的上过度解读聪明钱方向。

### 接口与隐私

- **链 ID**：BSC `56`、Base `8453`、Solana `CT_501`、Ethereum `1`（审计支持）。
- **与 `market-data-and-analysis.cn.md`**：榜单与宏观情绪用该文档中的 unified/social API；本 Task 偏地址级与信号级。
- **隐私**：响应中的钱包地址、持仓在展示给用户前按需脱敏。
