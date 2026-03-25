# rp-wallet-skill

RedotPay 支付签名 Skill — 为 Cursor Agent 提供 x402 支付签名能力。

## 包含的 Skill

| Skill | 目录 | 说明 |
|-------|------|------|
| `redotpay-web3-payment` | `web3-payment/` | EIP-3009 链上签名（用户钱包 / Agent 私钥） |
| `redotpay-balance-payment` | `balance-payment/` | RedotPay App 余额签名（设备授权凭证） |

> **只负责签名，不发送支付请求。** 签名结果返回给调用方（商户 Skill），由商户 Skill 发起最终的支付 GET 请求。

## 安装

### 1. 安装 MCP Server（前置依赖）

在 `~/.cursor/mcp.json` 中添加（如果文件不存在则新建）：

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

> npm 包地址：[rp-wallet-mcp](https://www.npmjs.com/package/rp-wallet-mcp)

### 2. 安装 Skill

```bash
npx skills add splcz/rp-wallet-skill -g -a cursor -y
```

或手动安装：

```bash
git clone https://github.com/splcz/rp-wallet-skill.git
cp -r rp-wallet-skill/web3-payment ~/.cursor/skills/redotpay-web3-payment
cp -r rp-wallet-skill/balance-payment ~/.cursor/skills/redotpay-balance-payment
```

安装后重启 Cursor（或 Reload Window）生效。

## 搭配使用

- **[rp-wallet-mcp](https://www.npmjs.com/package/rp-wallet-mcp)** — 签名 MCP Server（必需）
- **[rp-life-skill](https://github.com/splcz/rp-life-skill)** — 商户 Skill（如瑞幸咖啡），将签名委托给本 Skill

## License

MIT
