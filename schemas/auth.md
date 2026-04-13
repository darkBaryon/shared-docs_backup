# Auth Schema

## 登录方式

- 当前主链路：`wx.login()` 获取 `code`，调用 `/api/v1/auth/wechat_login` 换取系统 `token`。

## Token

- `token`：Bearer 访问令牌。
- 使用方式：`Authorization: Bearer <token>`。
- 适用范围：除登录类接口外的受保护接口。

## 过期机制

- token 视为短期有效。
- 过期后客户端应触发重新登录（当前契约优先保证可用性，后续可扩展 refresh token）。
