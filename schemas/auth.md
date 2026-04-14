# Auth Schema

## 当前认证策略

- 仅支持微信体系注册/登录。
- 不支持账号密码登录。
- 不支持手机号直接登录。

## 注册

- 接口：`POST /api/v1/auth/wechat/register`
- 请求参数：
  - `code`：来自 `wx.login()`
  - `phone_code`：来自 `wx.getPhoneNumber()`
- 约束：
  - 注册必须携带 `phone_code`，用于绑定当前微信授权手机号。
  - 未提供 `phone_code` 视为注册失败。

## 登录

- 接口：`POST /api/v1/auth/wechat_login`
- 请求参数：
  - `code`：来自 `wx.login()`
- 约束：
  - 仅已注册微信身份可登录。
  - 未注册用户需先调用注册接口。

## 身份映射

- `hs_usr_auth` 中维护两类身份绑定：
  - `identity_type=wechat` + `identifier=<openid>`
  - `identity_type=phone` + `identifier=<wechat_phone>`
- 两类身份通过同一个 `user_id` 关联到同一用户。

## Token

- `token`：Bearer 访问令牌。
- 使用方式：`Authorization: Bearer <token>`。
- 适用范围：除注册/登录接口外的受保护接口。
