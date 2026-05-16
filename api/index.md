# API

本目录维护各端接口契约和接口说明。

## 契约源

- 各前端端侧 API 文档是接口契约源。
- 当前已确认的小程序接口契约见 [miniapp-api.md](./miniapp-api.md)。
- [../overview/project-spec.md](../overview/project-spec.md)：项目通用规范，包含 API 路由格式和命名约束。

## 端侧文档

- [miniapp-api.md](./miniapp-api.md)：小程序接口文档。
- [publish.md](./publish.md)：发房系统接口文档。
- [admin.md](./admin.md)：管理后台接口文档。

## 通用文档

- [auth-flow.md](./auth-flow.md)：认证流程。
- [error-codes.md](./error-codes.md)：错误码。

## 规则

- 新增接口更新对应前端的端侧 API 文档。
- 不维护统一大而全的 OpenAPI 文件。
- 不维护历史 endpoint map。
- 接口路径统一采用 `POST /api/v{version}/{模块}/{动作}`。
- `{动作}` 必须是单个 path segment，不允许继续追加业务层级。
- 不为旧前端、旧数据库做兼容。
