# shared-docs

前后端共享契约文档仓库。这里维护 API、字段结构、交互流程与架构决策，是联调唯一参考源。

## 目录说明

- `api/`：接口定义与错误约定。
- `schemas/`：核心对象字段定义。
- `contracts/`：页面到接口的流程契约。

## Source of Truth

当前仓库的接口联调、字段含义、流程规范，统一以本目录为准。

## 建议阅读顺序

1. `api/openapi.yaml`
2. `api/error-codes.md`
3. `schemas/user.md`、`schemas/auth.md`、`schemas/house.md`、`schemas/dashboard.md`
4. `contracts/auth-flow.md`、`contracts/frontend-endpoint-map.md`、`contracts/backend-sync-changelog.md`
