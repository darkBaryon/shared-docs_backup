# API

本目录维护接口契约和接口说明。

## 契约源

- [openapi.yaml](./openapi.yaml)：API 契约源。
- [../overview/project-spec.md](../overview/project-spec.md)：项目通用规范，包含 API 路由格式和命名约束。

## 模块文档

- [miniapp.md](./miniapp.md)：小程序接口文档。
- [publish.md](./publish.md)：发房系统接口文档。
- [auth-flow.md](./auth-flow.md)：认证流程。
- [error-codes.md](./error-codes.md)：错误码。

## 规则

- 新增接口先更新 OpenAPI，再更新对应模块文档。
- 接口路径统一采用 `POST /api/v{version}/{模块}/{动作}`。
- `{动作}` 必须是单个 path segment，不允许继续追加业务层级。
- 不为旧前端、旧数据库做兼容。
