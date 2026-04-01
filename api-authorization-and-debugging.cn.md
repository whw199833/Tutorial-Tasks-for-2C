# API Authorization and Debugging（API 与系统授权配置）

## Description

**任务说明**：围绕 API Key 配置、授权交易、签名失败、子账户 API 与合约下单故障等「连接与权限」类问题，帮助定位是账户权限、密钥配置还是调用侧错误；在可自动化范围内配合账户与持仓查询。

**典型线上意图**：授权 AI 自主交易并设定盈利目标；API 密钥获取与配置指导；API 配置与调用失败；子账户 BNB 开期货与 API 问题；账户授权与签名调试；交易指令执行失败与签名问题；查询持仓并排查 API 授权。

**Skills 边界**：无单一 Skill 能「生成或修复」用户本地代码；`healthcheck` 偏环境与系统加固；`assets` / 衍生品 Skills 用于**验证**授权是否生效（能否读到余额/持仓/下单错误码语义）。`skill-vetter` 用于审查第三方 Skill 安装风险，与交易所 API 调试不同。

---

## 推荐 Skills 组合

| 角色 | Skill | 用途 |
|------|--------|------|
| 验证 | `assets` | 确认 API 或账户侧能否正确反映余额/权限相关状态 |
| 验证 | `derivatives-trading-usds-futures` / `derivatives-trading-coin-futures` | 持仓、订单错误场景与合约侧能力核对 |
| 验证 | `sub-account` | 子账户 API 与子账户结构是否匹配 |
| 环境 | `healthcheck` | 本地/运行环境安全与基础风险检查（非交易逻辑） |
| 扩展 | `skill-vetter` | 用户安装非官方 Skill 前的安全审查 |

---

## Plan（执行计划）

> 与 `Task_upgrade_advice.cn.md` §2 对齐：先归类错误 → 权限真值 → 分市场读通路 → 子账户/IP → 签名环境 → 供应链工具。

### A. 结构化流水线（DAG）

| 步骤 | 动作 |
|------|------|
| **归类** | 将报错映射到：`-1022` 签名、`-2015` Key、时间戳、IP、子账户 Key 错配、业务码；勿混谈。 |
| **权限真值** | 只读：`apiRestrictions`、`apiTradingStatus`、`account/status`。 |
| **读通路** | 现货 `GET /api/v3/account` → U 本位 `fapi/v2/account` 或 `positionRisk` → 币本位 `dapi/v1/account`；**失败层即问题层**。 |
| **子账户** | `sub-account/list` + `ipRestriction`（GET）核对 email ↔ Key ↔ IP。 |
| **签名/环境** | 时间同步、`recvWindow`、Secret；**P2P SAPI 参数不排序**。 |
| **供应链** | `skill-vetter` / `healthcheck` 与交易所验证**分结论**书写。 |

### B. 接口级速查

1. **验证 API Key 权限与交易开关（`assets`）**
   - `GET /sapi/v1/account/apiTradingStatus`：是否允许交易。
   - `GET /sapi/v1/account/apiRestrictions`：Key 权限范围。
   - `GET /sapi/v1/account/status`：账户状态。

2. **验证现货侧可读性（`spot`）**
   - `GET /api/v3/account`：若成功返回余额，说明现货 USER_DATA 可读。

3. **验证 U 本位合约侧（`derivatives-trading-usds-futures`）**
   - `GET /fapi/v2/account` 或 `GET /fapi/v3/account`：合约账户与持仓入口。
   - `GET /fapi/v2/positionRisk`：持仓；下单失败时对比 `GET /fapi/v1/apiTradingStatus`。

4. **验证币本位合约（`derivatives-trading-coin-futures`）**
   - `GET /dapi/v1/account`、`GET /dapi/v1/balance`：COIN-M 账户与余额。

5. **子账户 + IP 白名单（`sub-account`）**
   - `GET /sapi/v1/sub-account/list`：确认子账户邮箱。
   - `GET /sapi/v2/sub-account/subAccountApi/ipRestriction`：`email` + `subAccountApiKey`，查 IP 限制。
   - `POST` 同路径族：新增 IP 限制；`DELETE .../ipList`：删除 IP。

6. **`healthcheck` / `skill-vetter`**：按各自 SKILL 文档执行（不在本仓库 REST 表中展开）；与交易所 HTTP 错误码排查互补。

---

## 使用指南

### 结构化要点

- **一层失败只改一层**：例如 `dapi` 可读而 `fapi` 不可读 → 问题在期货权限或 URL，勿先改现货签名。
- **排查不写单**：默认只读验证；测试单仅作最后手段。

### 接口与错误码

- **错误码语义**：`-2015` 等需对照 Binance 官方错误码；`-1022` 签名失败时核对 HMAC、参数排序、`recvWindow`、本机时间同步。
- **P2P 类 SAPI 签名**：若涉及 `p2p` Skill 的 `/sapi/v1/c2c/...`，**不要对参数排序**（见 `p2p` SKILL 中 SAPI 说明），与标准 Spot/Futures 不同。
- **只读优先**：排查阶段避免 `POST` 下单；确需测试单可用 `spot` 的 `POST /api/v3/order/test`、`/fapi/v1/order/test`（按市场选择对应 Skill）。
- **与 `account-and-asset-management.cn.md` / `trading-execution.cn.md`**：权限正常后，资产查询走前者接口；交易走后者。
