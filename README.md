# rp-wallet-skill

RedotPay 支付签名 Skill — 为 Cursor Agent 提供 x402 支付签名能力。

## 包含的 Skill

| Skill | 目录 | 说明 |
|-------|------|------|
| `redotpay-web3-payment` | `web3-payment/` | EIP-3009 链上签名（用户钱包 / Agent 私钥） |
| `redotpay-balance-payment` | `balance-payment/` | RedotPay App 余额签名（设备授权凭证） |

> **只负责签名，不发送支付请求。** 签名结果返回给调用方（商户 Skill），由商户 Skill 发起最终的支付 GET 请求。

## 安装

将两个 Skill 复制到 `~/.cursor/skills/`：

```bash
# 克隆仓库
git clone https://github.com/splcz/rp-wallet-skill.git
cd rp-wallet-skill

# 复制 Skill
mkdir -p ~/.cursor/skills/redotpay-web3-payment
cp web3-payment/SKILL.md ~/.cursor/skills/redotpay-web3-payment/SKILL.md

mkdir -p ~/.cursor/skills/redotpay-balance-payment
cp balance-payment/SKILL.md ~/.cursor/skills/redotpay-balance-payment/SKILL.md
```

重启 Cursor（或 Reload Window）生效。

## 前置依赖

本 Skill 需要搭配 **rp-wallet-mcp** 使用。请先在 `~/.cursor/mcp.json` 中注册 MCP Server：

```json
{
  "mcpServers": {
    "redotpay-usdc": {
      "command": "npx",
      "args": ["-y", "rp-wallet-mcp"]
    }
  }
}
```

## 搭配使用

- **rp-wallet-mcp** — 提供 `sign_payment`、钱包管理、设备授权等 MCP 工具（必需）
- **rp-life-skill** — 商户 Skill（如瑞幸咖啡），负责创单和发支付请求，将签名委托给本 Skill

## License

MIT
