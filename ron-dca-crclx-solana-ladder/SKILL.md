---
name: ron-dca-crclx-solana-ladder
description: >-
  Ron's Solana ladder DCA for STRCx (mint Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH): when OKX USD price ≤ 100, buy 20% of the chosen stablecoin balance (USDT/USDC) per 0.5 USD grid step downward (L_n=100−0.5n). ONLY onchainOS for price and swaps plus okx-dex-market, okx-dex-swap, okx-dex-token, okx-security, okx-agentic-wallet. Triggers: STRCx ladder, 定投 STRCx, 定投 Xs78, 每跌0.5买余额20%, 100以下网格, Ron ladder DCA. Exclude DApp-named venues (okx-dapp-discovery), non-Solana mints, generic DCA without this mint.
license: MIT
metadata:
  author: ron-workspace
  version: "1.1.0"
  homepage: "https://web3.okx.com"
---

# Ron — STRCx Solana 价格网格定投（onchainOS）

> **符号说明**：链上/OKX DEX 聚合里该 mint 常显示为 **STRCx**（全名可能带 xStock 类描述）。标的 mint：`Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH`。

## Overview

一个可复用的 **Solana 网格定投策略 Skill**：当 STRCx（mint `Xs78…`) 的 OKX USD 价格跌到 100 及以下后，按 **每下跌 0.5 美元买入“账户稳定币余额的 20%”** 的阶梯逻辑执行买入，并用本地状态防止同一档重复成交。  

若用于 [Agentic 交易大赛 | Boost 交易赛](https://web3.okx.com/zh-hans/boost/trading-competition/agentic-trading)，支持“活动模式”合规约束与进度自查（自查不等于后台口径）。

## Core operations

- **Price**：`onchainos market price` 读取 STRCx USD 现价 \(P\)
- **Decide**：根据 \(L_n = 100 - 0.5n\) 与 `lastFilledN` 计算“下一档是否触发”
- **Execute**：用 `onchainos swap quote` → 用户确认 → `onchainos swap execute` 买入**余额 20%**
- **Persist**：写入 `.cursor/state/ron-dca-crclx-solana-ladder.json`（防重复买）
- **Contest safety**：活动模式下禁止循环/对冲刷量，并提示 ≥$100 资产维持要求

## Prerequisites

- `onchainos` CLI 已安装且可运行：`onchainos --version`
- Agentic Wallet 已登录：`onchainos wallet status` 显示 `loggedIn: true`
- Solana 钱包有：
  - **稳定币**（USDT 或 USDC）有余额（用于每档买入，按余额 20% 动态计算）
  - 少量 **SOL**（用于 gas）

## Quick Start（活动与非活动通用）

1. **检查登录与余额**
   - `onchainos wallet status`
   - `onchainos wallet balance --chain solana`（确认 USDT/USDC ≥ 50 与 SOL gas）
2. **查看当前价格**
   - `onchainos market price --address Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH --chain solana`
3. **若 \(P \le 100\)**：按本 skill 规则只触发 **下一档**（默认）：
   - `swap quote` → 显示 priceImpact / 风险项 → 用户确认
   - `swap execute`（本次买入额 = 余额 20%）
4. **写入状态**：更新 `lastFilledN`、`updatedAt`、`payToken`（见“状态”）

## Checklist（活动模式达标导向）

> 目标：在**合规**前提下，提高“有效交易量达标”的概率（最终以活动后台为准）。

- **参赛前**
  - 已报名参赛（通过 Agent 完成报名）
  - 钱包总资产**尽量长期 ≥ $100**（参与奖有不定时快照要求）
- **交易前**
  - 仅在 **Solana / X Layer** 上执行（本 skill 默认 Solana）
  - 避免稳定币互换、主网币/包装主网币互换作为“凑量”路径（活动明确有排除示例）
  - 不做“立刻买入再立刻卖回”的循环行为（禁止）
- **交易后**
  - 记录本地 `cumulativeNotionalUsd`（只做自查，不等于后台）
  - 定期检查是否进入榜单（`onchainos competition rank`；若接口不返回有效量字段则无法直接读到）

## 活动模式（用于 Agentic 交易大赛的合规达标）

> 活动页与规则参考：[Agentic 交易大赛 | Boost 交易赛](https://web3.okx.com/zh-hans/boost/trading-competition/agentic-trading)

<MUST>
当 Ron 明确表示“用于该活动并满足活动要求 / 达标 / 上榜 / 参与奖”等目标时，本 skill 必须启用 **活动模式**：

1. **只做合规交易**：禁止协助洗盘、循环交易、对冲交易、规避规则等行为；若用户要求“刷三次/立刻买卖回/对冲无风险暴露”等可疑操作，必须拒绝并说明活动风控可能取消资格。
2. **只计入指定链**：仅 **Solana** 与 **X Layer** 链上通过 Agentic Wallet 的代币交易才可能计入活动统计（以活动后台为准）。
3. **避免排除交易对**：活动说明明确：稳定币、主网币及 Wrapped 主网币之间的兑换不计入统计（例如 SOL-USDC、SOL-WSOL、USDT-USDC 等示例）。因此本策略在活动模式下仅围绕 **USDT/USDC ↔ STRCx** 执行，不做稳定币互换、不做 SOL↔WSOL 互换来“凑量”。
4. **上榜门槛**：PnL / PnL% 上榜需要比赛期间累计**有效交易量 ≥ 1,000 USD**（以活动后台口径为准）；参与奖还有“累计交易量 ≥ 100 USD 且钱包总资产全程 ≥ 100 USD”等要求与不定时快照。
5. **资产维持要求**：如目标包含参与奖，必须在执行前提醒用户确保钱包总资产**长期维持 ≥ 100 USD**，避免因转出/波动导致快照不达标。
</MUST>

<NEVER>
- 不要给出“如何规避风控/如何刷流水不被发现/如何对冲不产生风险暴露但刷量”等建议。
- 不要执行“立刻买入再立刻卖回同一资产”这类明显循环操作来刷交易量。
</NEVER>

## 策略规格（唯一权威）

| 字段 | 值 |
|------|-----|
| **链** | **Solana**（`solana` / `501`） |
| **标的 mint** | `Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH` |
| **价格源** | OKX / onchainOS：`onchainos market price --address <mint> --chain solana` 的 **USD 现价 `P`** |
| **激活条件** | **`P ≤ 100`** 后进入网格（未激活前只观察，不买入） |
| **网格步长** | **0.5 USD** |
| **每档买入** | **账户稳定币余额的 20%**（`exactIn`；USDT/USDC，6 位小数） |
| **网格线** | `L_n = 100 − 0.5 × n`，`n = 0,1,2,…` → 100, 99.5, 99, … |

**单档语义**：当 **`P ≤ L_n`** 且状态里 **`n` 尚未标记已成交**，则该档 **可触发一次**买入：金额 = 触发时刻所选稳定币余额的 **20%**；成交后写入 `lastFilledN ≥ n`（见 **状态**）。价格反弹后再跌回，**已成交档不重复买**（除非用户重置状态）。

<MUST>
**硬性要求**：读价与下单 **只能** 走 **onchainOS**（`onchainos`）及下方列出的 **OKX skills**。禁止用「只看 CoinGecko 就下单」替代；外链仅可作用户要求的 **交叉验证**，不得作为执行路径。
</MUST>

## 支付币种（用 USDT 还是 USDC）

1. 运行 `onchainos wallet balance --chain solana`。  
2. **默认**：若 **USDT 余额 ≥ 50** 且不低于 USDC，则用 **USDT**（`Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB`）；否则用 **USDC**（`EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`）。  
3. 若两者都 **< 50**：停止，提示充值；不要猜测跨链调资金。  
4. 用户 **明确指定** USDT 或 USDC 时，以用户为准（仍须余额足够）。

## Skill 路由

| 步骤 | Skill / 命令 |
|------|----------------|
| CLI 预检 | 读 `okx-agentic-wallet/_shared/preflight.md`；`export PATH="$HOME/.local/bin:$PATH"` |
| USD 价格 | `okx-dex-market` → `onchainos market price --address Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH --chain solana` |
| 代币信息 / 风险 | `okx-dex-token`；高敏感场景 `okx-security` |
| 换入标的 | `okx-dex-swap` → `swap quote` → 用户确认 → `swap execute` |
| 钱包 / Gas | `okx-agentic-wallet` |

## 执行流程（Agent 逐步）

1. **登录**  
   - `onchainos wallet status` → 已登录再继续。  

2. **读价**  
   - `onchainos market price --address Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH --chain solana`  
   - 解析 `P`；失败则展示原始 JSON，**不交易**。  

3. **`P > 100`**  
   - 策略空闲；可提示「高于 100，网格未激活」。**不** 自动清空状态。  

4. **`P ≤ 100`**  
   - 读取状态文件 `lastFilledN`（缺省 **-1**）。  
   - 集合 `S = { n ∈ ℕ | P ≤ L_n 且 n > lastFilledN }`。  
   - **默认（安全）**：只执行 **`min(S)`** 对应的一档：先 `swap quote`，再经用户确认后 `swap execute`（本次买入额 = 所选稳定币余额的 **20%**）→ mint；成功后 **`lastFilledN = min(S)`**。  
   - **补档（用户明确要求）**：按 `n` 升序列出待补档，**每一档单独 quote + 确认**；单次对话建议 **≤ 5 档**，更多需用户再次确认风险。  

5. **Swap**（遵守 `okx-dex-swap`）  
   - 首笔 **禁止** 擅自加 `--force`；81362 等需用户二次确认。  
   - 成交后按 skill 用语：**广播成功 ≠ 最终上链成功**；给 Solscan 等链接由用户核对。  

6. **写状态**  
   - 更新 `.cursor/state/ron-dca-crclx-solana-ladder.json` 的 `lastFilledN` 与 `updatedAt`（ISO8601）。  
   - 同时将本次名义成交额（= 本次实际 `--readable-amount`）累加到 `cumulativeNotionalUsd`（字符串形式保存，避免浮点误差）；更新 `lastTradeAt`。  

## 状态（防重复成交）

<MUST>
路径：`.cursor/state/ron-dca-crclx-solana-ladder.json`（目录已在仓库 `.cursor/state/.gitignore` 忽略 `*.json`）。  

**Schema：**
```json
{
  "lastFilledN": 0,
  "strategyVariant": "default",
  "payToken": "USDT",
  "contestMode": false,
  "cumulativeNotionalUsd": "0",
  "lastTradeAt": "2026-05-11T12:00:00Z",
  "updatedAt": "2026-05-11T12:00:00Z"
}
```
- `lastFilledN`：已完成的最高档索引 `n`；无成交为 **-1**。  
- `strategyVariant`：`default` = 含 **L₀=100** 档；若为 **`firstRung99p5`**，第一档从 **n=1（99.5）** 才开始买。  
- `payToken`：记录最近成交使用的 `USDT` 或 `USDC`，便于下次默认一致（仍须每次核对余额）。
- `contestMode`：是否以活动达标为目标（只影响风控与允许的交易形态；不改变网格触发公式）。
- `cumulativeNotionalUsd`：本 skill 记录的“名义成交额累计”（用于自查进度；**不等于**活动后台有效交易量）。
- `lastTradeAt`：最近一次广播交易时间，用于避免短时间内过密交易。
</MUST>

## 「跌到 100 之后」变体

- **默认**：`P ≤ 100` 时 **n=0** 档（100）可买。  
- 若 Ron 明确只要 **严格低于 100** 才买：设 `strategyVariant: "firstRung99p5"`，第一笔从 **n=1** 开始。

## OKX Agentic 大赛合规（必读提示）

若本策略用于 [Agentic 交易大赛](https://web3.okx.com/zh-hans/boost/trading-competition/agentic-trading) 相关场景，**不得**协助用户规避官方规则。活动说明示例：**稳定币与主网币/包装主网币之间兑换可能不计入统计**；**通过外部 DEX 对冲刷榜**等可能被风控取消资格。网格买 **USDT/USDC ↔ STRCx** 一般不属于「纯稳定↔主网币」排除描述，但 **是否计入有效交易量 / 收益** 以 **OKX 后台与活动条款** 为准。详见活动页「哪些交易会计入…」及条款中的禁止行为说明。

### 活动模式的执行节奏（降低异常交易风险）

<SHOULD>
在活动模式下，遵循以下节奏以保持交易行为更接近正常策略执行（非保证，只是风险更低）：

- **不做秒级回转**：单次买入后不要立刻卖回；至少跨过一段时间/价格变动再决定是否卖出（若用户坚持“立刻卖回刷流水”，必须拒绝）。
- **限制频率**：同一资产对在短时间内不要高频重复成交；以“触发一档 → 成交 → 记录状态”为一个自然事件。
- **透明记录**：每次成交更新 `cumulativeNotionalUsd` 与时间戳，方便用户对照活动门槛与自身交易行为是否合理。
</SHOULD>

### 进度自查（不代表活动后台）

活动模式下，在用户询问“离 1000U 还差多少 / 是否达标”时：

1. 读取 `cumulativeNotionalUsd` 给出本地估算值（说明不等于后台口径）。
2. 引导用户以活动页与后台为准，并可用 `onchainos competition rank`/`user-status` 做间接验证（若接口未提供有效交易量字段，则明确无法从 CLI 直接读取）。

## 风险

- 低流动性、高滑点、`isHoneyPot`、税费异常 → 按 `okx-dex-swap` **BLOCK/WARN** 处理。  
- 未授权 **静默自动交易** 前，每档都要有可追溯的用户确认（除非 Ron 已书面开启自动化并自担风险）。

## 反触发（不要用本 skill）

- 用户指定 **具体 DApp 场所**（Raydium/Uniswap 等）→ **`okx-dapp-discovery`**。  
- mint 或链与上表不一致 → 新策略，**禁止** 复用本状态文件。

## 示例

**例 A**：`P=99.2`，`lastFilledN=-1` → `S={0,1}`，默认只买 **n=0**（买入额=稳定币余额 20%），然后 `lastFilledN=0`。  

**例 B**：`P=98.9`，`lastFilledN=0` → 下一档 **n=1**（99.5），`98.9≤99.5` → 买入额=稳定币余额 20%，`lastFilledN=1`。  

**例 C**：`strategyVariant=firstRung99p5`，`P=99.8`，`lastFilledN=-1` → **n=0 跳过**，`S={1}`（若 `P≤99.5` 才含 1）；仅 `P≤99.5` 时买 **n=1**。

## 附录

- 命令与 mint 表：[reference.md](reference.md)

