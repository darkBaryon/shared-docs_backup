# Backend Auth

## 当前链路

- 微信登录 / 注册由 Go 后端 auth 模块处理。
- 登录成功后写入 Redis session。
- 客户端通过 `Authorization: Bearer <token>` 访问受保护接口。
- middleware 从 Redis 读取 session，并把用户信息写入 Gin context。

## 相关文档

- [../api/auth-flow.md](../api/auth-flow.md)
- [../schema/db-design/v4/modules/account-and-identity.md](../schema/db-design/v4/modules/account-and-identity.md)
