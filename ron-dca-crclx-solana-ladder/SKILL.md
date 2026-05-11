---
name: ron-dca-crclx-solana-ladder
description: >-
  Solana STRCx two-sided grid for mint Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH: buy 50 USDT per 0.5 down from 102, and sell ~50 USDT worth per 0.5 up after price >120.
license: MIT
metadata:
  author: ron-workspace
  version: "2.0.0"
  homepage: "https://github.com/ronliang521/strcx-grid-dca-skill"
---

# Ron — STRCx Solana 双向网格（onchainOS）

标的（Solana mint）：`Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH`（OKX/onchainOS 常显示符号 **STRCx**）。

## Triggers（触发词）

- `STRCx grid` / `STRCx 网格` / `Xs78 网格`
- `102开始每跌0.5买50U`
- `120以上每涨0.5卖50U`

## Exclusions（不要用本 skill）

- 用户指定具体 DApp 场所（Raydium/Uniswap 等）→ 走 `okx-dapp-discovery`
- 非 Solana、或 mint 不匹配 `Xs78...` → 新策略，禁止复用本状态文件

## Strategy spec（唯一权威）

### A. 买入网格（从 102 开始向下）

- **激活**：当 USD 现价 \(P \le 102\)
- **网格**：\(B_n = 102 - 0.5n\)，\(n=0,1,2,...\)
- **每档买入**：**50 USDT**（Solana USDT mint `Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB`）
- **停止**：钱包 **USDT < 50** 时停止继续买

### B. 卖出网格（120 以上后向上）

- **激活**：当 USD 现价 \(P > 120\)（满足“120 以上”条件后才启用卖出侧）
- **网格**：\(S_m = 120 + 0.5m\)，\(m=0,1,2,...\)
- **每档卖出**：卖出“**约等值 50 USDT**”的 STRCx  
  - 因 `onchainos swap execute` 为 **exactIn**，本策略按价格估算卖出数量：  
    \[
      sellTokenAmount = \left\lfloor \frac{50}{P} \right\rfloor_{8dp}
    \]
    其中 STRCx decimals 为 **8**，按 8 位小数向下取整。
- **停止**：STRCx 余额不足以卖出一档（`sellTokenAmount <= 0` 或余额不足）时停止继续卖

### 去重（同一档不重复成交）

- 买入：当 \(P \le B_n\) 且 \(n > buyLastFilledN\) 才能成交，成交后 `buyLastFilledN = n`
- 卖出：当 \(P \ge S_m\) 且 \(m > sellLastFilledM\) 才能成交，成交后 `sellLastFilledM = m`

## Core operations

- **Price**：`onchainos market price` 读取 USD 现价 \(P\)
- **Buy**：`swap quote` → 用户确认 → `swap execute`（USDT→STRCx，`--readable-amount "50"`）
- **Sell**：计算 `sellTokenAmount` → `swap quote` → 用户确认 → `swap execute`（STRCx→USDT，`--readable-amount "<sellTokenAmount>"`）
- **Persist**：写入状态文件防止重复成交（见下文）

## Prerequisites

- `onchainos --version`
- `onchainos wallet status` 显示已登录
- Solana 钱包：
  - 买入侧：USDT ≥ 50 + 少量 SOL gas
  - 卖出侧：STRCx 余额 + 少量 SOL gas

## Quick Start（plugin 风格）

1. `onchainos wallet status`
2. `onchainos wallet balance --chain solana`（确认 USDT、STRCx、SOL）
3. `onchainos market price --address Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH --chain solana`
4. 若 \(P \le 102\)：执行“下一档买入”（默认只做一档）
5. 若 \(P > 120\)：执行“下一档卖出”（默认只做一档）
6. 更新状态文件

## Contest mode（用于 Agentic 交易大赛的合规约束）

活动页参考：[Agentic 交易大赛 | Boost 交易赛](https://web3.okx.com/zh-hans/boost/trading-competition/agentic-trading)

<MUST>
若 Ron 明确目标是“参与活动并满足要求 / 达标 / 上榜 / 参与奖”，则必须启用活动合规约束：

- 禁止协助洗盘、循环交易、对冲交易、规避规则等；若用户要求“秒买秒卖回/刷 N 次/对冲无风险暴露”等，必须拒绝。
- 仅在活动允许链（Solana / X Layer）执行；本策略默认 Solana。
- 避免稳定币互换、主网币/包装主网币互换作为“凑量”路径（活动页有排除示例）。
</MUST>

## State（防重复成交）

状态文件：`.cursor/state/ron-strcx-solana-grid.json`

```json
{
  "buyLastFilledN": -1,
  "sellLastFilledM": -1,
  "payToken": "USDT",
  "contestMode": false,
  "cumulativeNotionalUsd": "0",
  "lastTradeAt": "2026-05-11T12:00:00Z",
  "updatedAt": "2026-05-11T12:00:00Z"
}
```

- `buyLastFilledN`：买入侧已成交最高档索引
- `sellLastFilledM`：卖出侧已成交最高档索引
- `cumulativeNotionalUsd`：本地自查名义额（每档 +50）；不等于活动后台口径

## Execution steps（Agent）

1. 读价：`onchainos market price ...` 取 \(P\)
2. 决策：
   - 若 \(P \le 102\)：计算 `S_buy`，只做下一档（默认）
   - 若 \(P > 120\)：计算 `S_sell`，只做下一档（默认）
3. Quote → 用户确认 → Execute（严格遵守 `okx-dex-swap` 的风险门槛；首笔不加 `--force`）
4. 成交后写状态

## Examples

- **买入例**：\(P=101.2\)，`buyLastFilledN=-1` → 下一档 `n=0`（102）满足 → 买 50 USDT；`buyLastFilledN=0`
- **卖出例**：\(P=121.1\)，`sellLastFilledM=-1` 且 \(P>120\) 激活 → 下一档 `m=0`（120）满足 → 卖出约 \(50/121.1\) STRCx（8 位小数向下取整）；`sellLastFilledM=0`

