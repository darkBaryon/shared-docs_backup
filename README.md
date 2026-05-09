# shared-docs

前后端共享文档仓库。这里维护 API、变更记录、数据库结构与后端实现约定，是联调唯一参考源。

## 目录说明

- `api/`：接口文档、接口映射、前后端联调说明。
- `changes/`：口径调整、同步记录、评审检查记录。
- `schema/`：数据库设计文档与后端 CRUD 实现约定。

### 迁移专题

- `changes/migration/project-background.md`：项目背景、前后端组成和迁移关系。
- `changes/migration/index.md`：迁移背景、范围与文档总目录。
- `changes/migration/current-status.md`：当前已完成事项与阶段判断。
- `changes/migration/next-steps.md`：后续实施计划与推进顺序。

## Source of Truth

当前仓库的接口联调、字段含义、流程规范，统一以本目录为准。

## 建议阅读顺序

1. `api/openapi.yaml`
2. `api/error-codes.md`
3. `api/auth-flow.md`、`api/miniapp接口文档.md`、`api/发房系统接口文档.md`
4. `changes/backend-sync-changelog.md`
5. `schema/db-design/v4/README.md`
6. `schema/backend-crud.md`
7. `changes/migration/index.md`
