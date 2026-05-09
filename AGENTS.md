# AGENTS

## 作用范围

本规则适用于 `shared-docs/` 目录下全部文件与子目录。

## 仓库定位

这是共享文档仓库，不承载实现代码。

## 最高约束

1. 本项目是全新项目，禁止任何兼容性设计。
2. 数据模型只服务当前确认需求，不为历史版本、旧前端、旧数据库做适配。
3. Python 项目只作为业务行为参考，不作为 Go 技术架构模板。
4. 有疑问先读 shared-docs，不要猜。

## 文档模块

- `agents/`：不同端开发 Agent 的工作规则。
- `overview/`：项目背景、系统组成、术语。
- `product/`：产品范围和业务边界。
- `api/`：接口契约、接口说明、错误码、认证流程。
- `schema/`：数据库设计、字段约束、索引、后端 CRUD。
- `backend/`：Go 后端工程约定。
- `frontend/`：前端工程约定和接口映射。
- `flows/`：跨模块业务流程。
- `changes/`：变更记录、迁移进度、架构决策。

## 契约优先级

1. API 路径、方法、请求响应：以 `api/openapi.yaml` 为准。
2. 接口补充说明与映射：以 `api/` 下文档为准，并与 OpenAPI 保持同步。
3. 数据库结构、枚举、索引、后端 CRUD：以 `schema/` 为准。
4. 工程边界和实现约定：以 `backend/`、`frontend/`、`agents/` 为准。
5. 历史调整说明与同步记录：以 `changes/` 为准。

## 编辑规则

1. 文件名使用英文 kebab-case，正文可以使用中文。
2. `README.md` 只做总入口，不承载大量正文。
3. 每个顶层目录应有 `index.md`。
4. 规范类文档是当前事实源；接口变更必须同时更新：
   - `api/openapi.yaml`
   - `api/` 下对应接口说明文档
5. 数据库变更必须同时更新：
   - `schema/db-design/`
   - `schema/backend-crud/`
6. 若当前实现或最终策略与历史输入存在差异：
   - 优先新增或更新 `changes/decisions/`。
   - 必要时在 `changes/` 下新增日期化说明文档。
7. 链接使用相对路径。

## 交付检查清单

- [ ] 新增或迁移文件后，相关 `index.md` 已更新
- [ ] API 变更已同步 `api/openapi.yaml`
- [ ] schema 变更已同步数据库设计与 CRUD 文档
- [ ] 架构口径变更已同步 `changes/decisions/`
- [ ] 旧路径引用已清理
