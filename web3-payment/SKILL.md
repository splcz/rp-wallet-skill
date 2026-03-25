---
name: redotpay-web3-payment
description: Complete x402 Web3 (eip155) payment signing with EIP-3009. Use when the merchant skill delegates chain payment signing (provides paymentRequiredBase64 + acceptIndex), or when the user says "设置钱包", "删除钱包", "查看钱包", "更换私钥" for wallet management.
---

# RedotPay Web3 支付签名

通过 x402 协议为 EVM 链上的 USDC 支付生成 EIP-3009 TransferWithAuthorization 签名。

**本 skill 只负责签名，不发送支付请求。** 签名完成后将结果返回给调用方（商户 skill），由调用方发起最终的支付 GET 请求。

## 触发条件

### 签名（由商户 skill 委托）
当商户 skill 下单后用户选择了 eip155 网络支付，会提供：
- **paymentRequiredBase64** — 第 2.2 步获取的 Base64 编码 payment-required JSON
- **acceptIndex** — 用户选择的支付方式索引

### 钱包管理（直接触发）
- "设置钱包"、"设置私钥"、"更换私钥" → 调用 `set_agent_wallet`
- "删除钱包"、"删除私钥"、"移除钱包" → 调用 `remove_agent_wallet`
- "查看钱包"、"钱包状态" → 调用 `get_agent_wallet`

## 签名流程

### 第一步：选择签名方式

**必须使用 AskQuestion 工具弹出选择框：**

```
AskQuestion:
  title: "选择签名方式"
  questions:
    - id: "signer"
      prompt: "请选择签名方式"
      options:
        - id: "user"
          label: "用户签名 — 打开浏览器，通过 MetaMask 或 WalletConnect 签名"
        - id: "agent"
          label: "Agent 签名 — 使用已存储的私钥自动签名（无需人工操作）"
```

### 第二步：执行签名

#### 用户选择"用户签名"

直接调用 `sign_payment`：

```
CallMcpTool: redotpay-usdc → sign_payment
Arguments: { "paymentRequiredBase64": "...", "acceptIndex": 0, "signer": "user" }
```

会打开浏览器签名页面，用户通过钱包签名。

#### 用户选择"Agent 签名"

先检查是否有存储的钱包：

```
CallMcpTool: redotpay-usdc → get_agent_wallet
```

- **如果已有钱包**：直接调用签名

```
CallMcpTool: redotpay-usdc → sign_payment
Arguments: { "paymentRequiredBase64": "...", "acceptIndex": 0, "signer": "agent" }
```

- **如果没有钱包**：通过对话让用户提供私钥，然后设置钱包

```
CallMcpTool: redotpay-usdc → set_agent_wallet
Arguments: { "private_key": "用户提供的私钥" }
```

设置成功后，再调用 `sign_payment`。

### 第三步：返回签名结果

`sign_payment` 成功后会返回：

```json
{
  "paymentSignatureBase64": "Base64 编码的 payment-signature",
  "paymentUrl": "支付请求目标 URL",
  "network": "eip155:8453",
  "signerMode": "Agent 自动签名"
}
```

**将以上完整结果返回给调用方（商户 skill）。** 本 skill 不发送支付请求。

## 钱包管理工具

### 设置钱包
```
CallMcpTool: redotpay-usdc → set_agent_wallet
Arguments: { "private_key": "0x..." }
```
私钥安全存储在 `~/.redotpay/`，文件权限 0600。

### 查看钱包
```
CallMcpTool: redotpay-usdc → get_agent_wallet
```

### 删除钱包
```
CallMcpTool: redotpay-usdc → remove_agent_wallet
```

## 结果处理

- **签名成功**：返回 `paymentSignatureBase64` 等字段给调用方
- **超时**：提示用户未在 5 分钟内完成签名（仅用户签名模式）
- **错误**：展示错误信息
