# Reference — STRCx 双向网格（Solana）

## 常量

| 含义 | 地址 / 值 |
|------|-----------|
| 标的 mint（STRCx） | `Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH` |
| 链 | `solana` 或 `501` |
| Solana USDT | `Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB` |
| 买入网格 | `B_n = 102 - 0.5*n`，`n ≥ 0` |
| 卖出网格 | `S_m = 120 + 0.5*m`，`m ≥ 0`（仅当价格 > 120 才激活卖出侧） |
| 每档买入 | `50`（USDT，`--readable-amount "50"`） |
| 每档卖出 | 约等值 `50` USDT 的 STRCx：`sellTokenAmount = floor_to_8dp(50 / P)` |

## 命令模板

```bash
export PATH="$HOME/.local/bin:$PATH"

# 现价（USD）
onchainos market price --address Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH --chain solana

# 余额（USDT、STRCx、SOL gas）
onchainos wallet balance --chain solana
```

**买入（USDT → STRCx，每档 50 USDT）：**

```bash
onchainos swap quote \
  --from Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB \
  --to Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH \
  --readable-amount "50" \
  --chain solana

onchainos swap execute \
  --from Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB \
  --to Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH \
  --readable-amount "50" \
  --chain solana \
  --wallet "<你的 Solana 地址>"
```

**卖出（STRCx → USDT，每档约等值 50 USDT）：**

1) 读价得到 `P`（USD）  
2) 计算 `sellTokenAmount = floor_to_8dp(50 / P)`（STRCx 8 位小数向下取整）  
3) 执行：

```bash
onchainos swap quote \
  --from Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH \
  --to Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB \
  --readable-amount "<sellTokenAmount>" \
  --chain solana

onchainos swap execute \
  --from Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH \
  --to Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB \
  --readable-amount "<sellTokenAmount>" \
  --chain solana \
  --wallet "<你的 Solana 地址>"
```

## 状态文件

- 路径：`.cursor/state/ron-strcx-solana-grid.json`  
- 字段：`buyLastFilledN`、`sellLastFilledM`、`payToken`、`contestMode`、`cumulativeNotionalUsd`、`updatedAt`（见 `SKILL.md`）

## 活动链接

- [Agentic 交易大赛 | Boost 交易赛](https://web3.okx.com/zh-hans/boost/trading-competition/agentic-trading)

