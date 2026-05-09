# shared-docs

前后端共享文档仓库。这里维护项目背景、产品范围、API、数据库结构、后端约定、前端约定、跨端流程、Agent 指令和变更记录，是联调与迁移工作的共同事实源。

## 目录

- [agents/](./agents/index.md)：不同端开发 Agent 的工作规则。
- [overview/](./overview/index.md)：项目全貌、系统组成、术语。
- [product/](./product/index.md)：小程序、发房系统、后台管理、AI chat 的产品范围。
- [api/](./api/index.md)：各端接口文档、错误码、认证流程。
- [schema/](./schema/index.md)：数据库设计与后端 CRUD 实现约定。
- [backend/](./backend/index.md)：Go 后端工程架构与实现约定。
- [frontend/](./frontend/index.md)：前端工程与端侧约定。
- [flows/](./flows/index.md)：跨模块业务流程。
- [changes/](./changes/index.md)：迁移进度、变更记录、架构决策。

## Source of Truth

1. 项目通用规范、API 命名、MongoDB / Redis 约定：先看 [overview/project-spec.md](./overview/project-spec.md)。
2. API 路径、方法、请求响应：优先看 [api/](./api/index.md) 下对应前端的端侧 API 文档。
3. API 通用命名规则和模块口径：看 [overview/project-spec.md](./overview/project-spec.md) 与 [api/](./api/index.md)。
4. 数据库字段、枚举、索引、约束：看 [schema/](./schema/index.md)。
5. 后端实现边界：看 [backend/](./backend/index.md)。
6. 不同端 Agent 接手任务：先看 [agents/](./agents/index.md)。
7. 历史调整、迁移进度、架构决策：看 [changes/](./changes/index.md)。

## 建议阅读顺序

1. [overview/project-background.md](./overview/project-background.md)
2. [overview/project-spec.md](./overview/project-spec.md)
3. [overview/system-map.md](./overview/system-map.md)
4. [agents/global.md](./agents/global.md)
5. 对应端的 Agent 文档，例如 [agents/backend-go.md](./agents/backend-go.md)
6. 对应前端 API，例如 [api/miniapp-api.md](./api/miniapp-api.md)
7. 对应数据库设计，例如 [schema/db-design/v4/modules/house-master-data.md](./schema/db-design/v4/modules/house-master-data.md)
8. [changes/migration/current-status.md](./changes/migration/current-status.md)

当前小程序 HPD 规划见 [backend/miniapp-hpd.md](./backend/miniapp-hpd.md)。
