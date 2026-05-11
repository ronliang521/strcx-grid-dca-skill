# Reference — STRCx / CRCLx 网格定投（Solana）

## 常量

| 含义 | 地址 / 值 |
|------|-----------|
| 标的 mint（STRCx / 你称 CRCLx） | `Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH` |
| 链 | `solana` 或 `501` |
| Solana USDT | `Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB` |
| Solana USDC | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` |
| 网格 | `L_n = 100 - 0.5*n`，`n ≥ 0` |
| 每档名义 | `50`（`--readable-amount "50"`） |

## 命令模板

```bash
export PATH="$HOME/.local/bin:$PATH"

# 现价（USD）
onchainos market price --address Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH --chain solana

# 余额（选 USDT 或 USDC）
onchainos wallet balance --chain solana
```

**用 USDT 买 50U 标的：**

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

**用 USDC 时**：把 `--from` 换成 `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` 即可。

## 状态文件

- 路径：`.cursor/state/ron-dca-crclx-solana-ladder.json`  
- 字段：`lastFilledN`、`strategyVariant`、`payToken`、`updatedAt`（见 `SKILL.md`）

## 活动链接

- [Agentic 交易大赛 | Boost 交易赛](https://web3.okx.com/zh-hans/boost/trading-competition/agentic-trading)

