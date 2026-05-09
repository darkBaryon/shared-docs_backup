# Auth Implementation And Adjustment Notes (2026-04-14)

本文用于说明：
- 基于原始 `api/miniapp.md` 的接口实现情况
- 本次注册/登录策略调整内容与落地状态

## 1. 基于原始需求文档的实现情况

原始文档中“认证相关（新增）”要求过以下接口：
- `POST /api/v1/auth/login/phone`
- `POST /api/v1/auth/login/password`
- `POST /api/v1/auth/register/phone`
- `POST /api/v1/auth/register/password`

本轮开发中，上述接口均已在代码中实现过，并完成了：
- 统一响应结构 `code/message/data`
- `data.token` 返回
- 用户资料更新接口 `POST /api/v1/user/profile/update`

随后，经产品/业务策略确认，认证体系口径变更为“仅微信体系”，故上述四个接口已下线。

## 2. 本次注册/登录逻辑调整（最终口径）

### 2.1 当前有效认证流程

仅保留以下两条认证接口：
1. `POST /api/v1/auth/wechat/register`
2. `POST /api/v1/auth/wechat_login`

### 2.2 注册（强制绑定微信手机号）

`POST /api/v1/auth/wechat/register`

请求体：
```json
{
  "code": "wx-login-code",
  "phone_code": "wx-phone-code"
}
```

后端处理：
- 使用 `code` 换取 `openid`
- 使用 `phone_code`（通过微信服务端接口）换取手机号
- 注册时同时绑定：
  - `identity_type=wechat` + `identifier=openid`
  - `identity_type=phone` + `identifier=<wechat-phone>`
- 返回 `token`

约束：
- 未提供 `phone_code` 视为注册不完整，直接失败
- 不再支持短信验证码绑定流程

### 2.3 登录（仅已注册用户）

`POST /api/v1/auth/wechat_login`

请求体：
```json
{
  "code": "wx-login-code"
}
```

后端处理：
- 用 `code` 换 `openid`
- 仅当 `openid` 已存在绑定关系时允许登录
- 未注册用户返回可读错误（需先走注册接口）

## 3. 数据模型说明（hs_usr_auth）

认证关系仍使用 `hs_usr_auth`：
- `identity_type=wechat` 记录微信身份
- `identity_type=phone` 记录微信手机号身份
- 两条记录通过同一个 `user_id` 归并到同一用户

## 4. OpenAPI 同步

本次已同步更新：
- `api/openapi.yaml`

同步点：
- 新增 `POST /api/v1/auth/wechat/register`
- 保留并说明 `POST /api/v1/auth/wechat_login` 为“仅已注册可登录”
- 移除旧的通用登录入口 `/api/v1/auth/login`
- 补充 `POST /api/v1/user/profile/update` 契约

## 5. 说明

`api/miniapp.md` 作为阶段性需求输入文档，保持原始记录不改写；
本文件用于沉淀“已实现结果 + 口径调整”作为交付说明。
