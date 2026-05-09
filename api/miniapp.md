# miniapp 接口文档

本文件用于记录小程序前端当前已接入、需要后端实现或确认的接口契约。

## 1) 认证相关（新增）

### `POST /api/v1/auth/login/phone`
- 用途：手机号登录
- 请求体：
```json
{
  "phone": "13800138000",
  "sms_code": "123456"
}
```
- 成功响应（示例）：
```json
{
  "code": 0,
  "message": "ok",
  "data": {
    "token": "jwt-token"
  }
}
```

### `POST /api/v1/auth/login/password`
- 用途：账号密码登录
- 请求体：
```json
{
  "account": "test_user",
  "password": "123456"
}
```
- 成功响应：同上，`data.token` 必填。

### `POST /api/v1/auth/register/phone`
- 用途：手机号注册（注册成功后前端自动登录）
- 请求体：
```json
{
  "phone": "13800138000",
  "sms_code": "123456"
}
```
- 成功响应：同上，`data.token` 必填。

### `POST /api/v1/auth/register/password`
- 用途：账号密码注册（注册成功后前端自动登录）
- 请求体：
```json
{
  "account": "test_user",
  "password": "123456"
}
```
- 成功响应：同上，`data.token` 必填。

## 2) 用户资料相关（新增）

### `POST /api/v1/user/profile/update`
- 用途：个人页编辑资料（昵称、头像）
- 鉴权：`Authorization: Bearer <token>`
- 请求体：
```json
{
  "nickname": "小明",
  "avatar": "https://example.com/avatar.jpg"
}
```
- 成功响应（建议返回最新资料）：
```json
{
  "code": 0,
  "message": "ok",
  "data": {
    "nickname": "小明",
    "avatar": "https://example.com/avatar.jpg"
  }
}
```

## 3) 已在使用的依赖接口（需继续保持）

- `POST /api/v1/auth/wechat_login`
- `POST /api/v1/user/profile`
- `POST /api/v1/user/dashboard`
- `POST /api/v1/house/public_detail`
- `POST /api/v1/favorite/add`
- `POST /api/v1/favorite/remove`
- `POST /api/v1/favorite/list`
- `POST /api/v1/history/add`
- `POST /api/v1/history/list`

## 4) 通用约定

- 响应结构统一：`{ code, message, data }`
- 前端按 `code === 0` 视为业务成功
- 认证接口以外，默认需要 Bearer Token
- 失败时建议返回 `message` 或 `detail`，便于前端直接提示
