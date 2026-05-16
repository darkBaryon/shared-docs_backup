# Go 后端工作流状态

本文只保留 Go 后端当前状态和任务入口。具体实施计划放在 `active/`，长期规范沉淀到 `backend/`、`api/`、`schema/`、`flows/`。

## 当前事实

后端按“端侧 HTTP 表达层 -> 端侧应用服务 -> 内部领域能力 -> repository”组织：

```text
handler/v1/{terminal}/{module}
  -> service/{terminal}/{module}
    -> domain/{capability}
      -> repository/{data-subdomain}
        -> MongoDB / Redis
```

已确认的内部领域：

- `internal/domain/hmd`：房源主数据领域能力。
- `internal/domain/listingprojection`：read model 投影。
- `internal/domain/publishaccess`：发房归属关系与后续数据作用域基础。

已确认的端侧应用服务：

- `internal/service/miniapp/auth`
- `internal/service/miniapp/house`
- `internal/service/publish`

其中 `internal/service/miniapp/auth` 内部直接承载微信登录、用户初始化、资料活跃时间维护，不再单独拆 `domain/miniappauth`。

已确认的 repository 命名依据：

- 按底层数据子域分包。
- repository 包名优先表达实体/collection 语义，例如 `landlord`、`hmd`、`hpd`。

## 已完成

- 基础设施：配置读取、Mongo、Redis、Redis session token、repository common 基础能力。
- 小程序 auth：`wechat_login`、`wechat_register`、`session`。
- Publish 第一阶段：HMD 写链路、HPD projector、真实服务联调。
- HPD 小程序展示层：`search`、`public_detail`。
- 小程序用户资料、收藏、足迹：profile、dashboard、favorite、history。
- 小程序 `public_detail` 可选登录：匿名返回 `is_favorited=false`，有效 token 返回真实收藏状态，无效 token 按匿名处理。

## 当前任务

- [Publish Auth 与数据作用域计划](./active/publish-auth-data-scope.md)：阶段 2 前端登录态接入已实现并 review 通过，待阶段 3 后端 root owner scope relation。
- [后端业务流日志计划](./active/business-flow-logging.md)
- [管理端 API 实施计划](./active/admin-api.md)
- [小程序后续模块计划](./active/miniapp-followups.md)

## 必读文档

1. [../../api/miniapp-api.md](../../api/miniapp-api.md)：小程序 API 契约。
2. [../../api/publish.md](../../api/publish.md)：发房端 API 契约。
3. [../../backend/architecture.md](../../backend/architecture.md)：Go 后端分层。
4. [../../backend/publish.md](../../backend/publish.md)：publish 后端链路。
5. [../../backend/miniapp-hpd.md](../../backend/miniapp-hpd.md)：小程序 HPD 展示层。
6. [../../schema/db-design/v4/index.md](../../schema/db-design/v4/index.md)：数据库设计入口。

## 维护规则

- 新增或完成后端任务时，更新本文的“已完成”和“当前任务”。
- 具体实施步骤写入 `active/`，不要塞回本文。
- review report 写入 `reviews/YYYY-MM-DD-topic.md`。
- 已确认的接口、字段、架构口径必须同步到长期规范文档。
