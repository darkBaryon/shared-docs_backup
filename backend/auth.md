# Backend Auth

## 当前链路

- 小程序微信登录 / 注册由 `handler/v1/miniapp/auth` 和 `service/miniapp/auth` 处理。
- 微信身份绑定、用户初始化等内部能力位于 `domain/auth`。
- 登录成功后写入 Redis session。
- 客户端通过 `Authorization: Bearer <token>` 访问受保护接口。
- middleware 从 Redis 读取 session，并把用户信息写入 Gin context。

约束：

- 小程序 auth 是端侧应用服务，不作为 publish/admin 的通用登录入口。
- 后续 publish/admin auth 应分别建立自己的 `service/{terminal}/auth`。

## 相关文档

- [../api/auth-flow.md](../api/auth-flow.md)
- [../schema/db-design/v4/modules/account-and-identity.md](../schema/db-design/v4/modules/account-and-identity.md)
