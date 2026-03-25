---
name: redotpay-balance-payment
description: Complete RedotPay in-app balance payment signing after device authorization. Use when the merchant skill delegates a redotpay:balance payment signing (provides paymentRequiredBase64 + acceptIndex), or when the user mentions "余额支付"、"RedotPay 余额"、"扫码授权"、"设备授权"、"切换 RedotPay 账户"、"重新授权".
---

# RedotPay 余额支付签名

通过 RedotPay App 设备授权后的本地凭证，为 **redotpay:balance** 支付生成签名。

**本 skill 只负责授权和签名，不发送支付请求。** 签名完成后将结果返回给调用方（商户 skill），由调用方发起最终的支付 GET 请求。

## 触发条件

### 签名（由商户 skill 委托）

商户 skill 下单后，用户选择了 **RedotPay 余额支付**（`accepts[i].network === "redotpay:balance"`）。此时应提供：

- **paymentRequiredBase64** — Base64 编码的 payment-required JSON
- **acceptIndex** — 该支付方式在 `accepts` 数组中的索引

Agent 阅读并执行本 skill，完成授权检查与签名，返回签名结果。

### 授权与账户（用户直接提及）

- 「扫码授权」「设备授权」「RedotPay 授权」→ 按下方 **设备未授权** 流程处理（`check_authorization` → 展示二维码 → `confirm_authorization`）
- 「切换 RedotPay 账户」「重新授权」→ 使用 **reset_authorization** 小节

## 签名流程

### 第一步：检查设备授权

调用 `redotpay-usdc` MCP 服务器的 `check_authorization` 工具（无参数）。

- **如果返回表示设备已授权**：直接进行第二步签名。

- **如果返回二维码内容**：设备需要授权。
  1. 将二维码图片展示给用户，提示使用 RedotPay App 扫码授权。
  2. **立即**调用 `redotpay-usdc` MCP 服务器的 `confirm_authorization` 工具（无需等待用户回复），工具会自动轮询服务端直到用户完成扫码或超时：

```json
{ "qrCode": "check_authorization 返回的二维码内容" }
```

  3. 若返回授权成功，继续第二步签名。
  4. 若返回超时或已过期，提示用户并重新调用 `check_authorization` 获取新二维码。

### 第二步：执行签名

调用 `redotpay-usdc` MCP 服务器的 `sign_payment` 工具（**不传** `signer` 参数，由 MCP 使用已授权的凭证密钥签名）：

```json
{ "paymentRequiredBase64": "...", "acceptIndex": 1 }
```

### 第三步：返回签名结果

`sign_payment` 成功后会返回：

```json
{
  "paymentSignatureBase64": "Base64 编码的 payment-signature",
  "paymentUrl": "支付请求目标 URL",
  "network": "redotpay:balance",
  "signerMode": "RedotPay 授权凭证签名"
}
```

**将以上完整结果返回给调用方（商户 skill）。** 本 skill 不发送支付请求。

## 切换账户 / 重新授权

调用 `redotpay-usdc` MCP 服务器的 `reset_authorization` 工具（无参数），清除本地旧密钥与凭证并生成新二维码。

将返回的新二维码展示给用户后，**立即**调用 `confirm_authorization`（传入本次获得的 `qrCode`）自动轮询等待授权完成。

## 结果处理

- **签名成功**：返回 `paymentSignatureBase64` 等字段给调用方
- **授权过期**：提示用户重新扫码授权
- **错误**：展示错误信息，可按需重试授权或签名步骤
