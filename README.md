# rp-wallet-skill

RedotPay 支付签名 Skill — 为 AI Agent 提供 x402 支付签名能力。

## 支持的 Agent

| Agent | 状态 |
|-------|------|
| Cursor | ✅ 已验证 |
| OpenClaw | ✅ 已适配 |
| Claude Code | ✅ 兼容 |

## 包含的 Skill

| Skill | 目录 | 说明 |
|-------|------|------|
| `redotpay-web3-payment` | `web3-payment/` | EIP-3009 链上签名（用户钱包 / Agent 私钥） |
| `redotpay-balance-payment` | `balance-payment/` | RedotPay App 余额签名（设备授权凭证） |

> **只负责签名，不发送支付请求。** 签名结果返回给调用方（商户 Skill），由商户 Skill 发起最终的支付 GET 请求。

## 安装

### 1. 安装 MCP Server（前置依赖）

将 `redotpay-usdc` MCP 服务器添加到你的 Agent 配置中：

<details>
<summary>Cursor</summary>

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
</details>

<details>
<summary>OpenClaw</summary>

在 `~/.openclaw/openclaw.json` 中添加：

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
</details>

> npm 包地址：[rp-wallet-mcp](https://www.npmjs.com/package/rp-wallet-mcp)

### 2. 安装 Skill

**一键安装（推荐）：**

```bash
# Cursor
npx skills add splcz/rp-wallet-skill -g -a cursor -y

# OpenClaw
npx skills add splcz/rp-wallet-skill -g -a openclaw -y

# 所有已安装的 Agent
npx skills add splcz/rp-wallet-skill -g --all -y
```

**手动安装：**

```bash
git clone https://github.com/splcz/rp-wallet-skill.git

# Cursor
cp -r rp-wallet-skill/web3-payment ~/.cursor/skills/redotpay-web3-payment
cp -r rp-wallet-skill/balance-payment ~/.cursor/skills/redotpay-balance-payment

# OpenClaw
cp -r rp-wallet-skill/web3-payment ~/.openclaw/skills/redotpay-web3-payment
cp -r rp-wallet-skill/balance-payment ~/.openclaw/skills/redotpay-balance-payment
```

安装后重启 Agent 生效。

## 搭配使用

- **[rp-wallet-mcp](https://www.npmjs.com/package/rp-wallet-mcp)** — 签名 MCP Server（必需）
- **[rp-life-skill](https://github.com/splcz/rp-life-skill)** — 商户 Skill（如瑞幸咖啡），将签名委托给本 Skill

## License

MIT
