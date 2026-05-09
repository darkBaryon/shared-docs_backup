# Frontend Miniapp Agent

## 必读

1. [global.md](./global.md)
2. [../product/miniapp.md](../product/miniapp.md)
3. [../api/miniapp-api.md](../api/miniapp-api.md)
4. [../api/auth-flow.md](../api/auth-flow.md)
5. [../frontend/index.md](../frontend/index.md)

## 工作规则

- 接口路径、字段、错误码以 `api/` 为准。
- 登录态使用 Go 后端返回的 Bearer token。
- 不为旧接口或旧字段做兼容。
- 页面字段展示必须和 API / schema 口径一致。
- 修改接口调用封装时，同步 [../api/miniapp-api.md](../api/miniapp-api.md)。
