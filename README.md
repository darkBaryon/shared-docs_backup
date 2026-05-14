# shared-docs

这是项目共享文档仓库。目标是让前端、后端和 Agent 能快速知道：

- 当前项目做到哪里了
- 接口契约是什么
- 数据库字段是什么
- Go 后端代码应该按什么边界实现
- 接下来该做什么

## 先看这些文件

大多数任务先读这几个就够了：

1. [workstreams/index.md](./workstreams/index.md)：各端当前推进入口。
2. [workstreams/backend-go/status.md](./workstreams/backend-go/status.md)：Go 后端当前状态和下一步。
3. [workstreams/publish-web/status.md](./workstreams/publish-web/status.md)：出房 Web 当前状态和下一步。
4. [workstreams/miniapp/status.md](./workstreams/miniapp/status.md)：小程序前端当前状态和下一步。
5. [api/miniapp-api.md](./api/miniapp-api.md)：小程序接口契约。
6. [api/publish.md](./api/publish.md)：发房端接口契约。
7. [backend/architecture.md](./backend/architecture.md)：Go 后端分层和代码边界。
8. [schema/db-design/v4/index.md](./schema/db-design/v4/index.md)：数据库设计入口。
9. [frontend/development-spec.md](./frontend/development-spec.md)：前端目标架构、分层和目录规范。

具体任务计划不要塞进 `status.md`，应放在对应端的 `workstreams/{app}/active/`。
Review report 放在对应端的 `workstreams/{app}/reviews/`。

## 当前后端形态

Go 后端当前按“端侧应用服务 + 内部领域能力”组织：

```text
handler/v1/{terminal}/{module}
  -> service/{terminal}/{module}
    -> domain/{capability}
      -> repository/{data-module}
        -> MongoDB / Redis
```

当前已经落地的关键路径：

```text
handler/v1/miniapp/auth
  -> service/miniapp/auth
    -> domain/auth

handler/v1/miniapp/house
  -> service/miniapp/house
    -> repository/hpd

handler/v1/publish
  -> service/publish
    -> domain/hmd
    -> domain/hpd
```

## 当前接口状态

已接入：

- 小程序认证：
  - `POST /api/v1/auth/wechat_login`
  - `POST /api/v1/auth/wechat_register`
  - `POST /api/v1/auth/session`
- 小程序找房：
  - `POST /api/v1/house/search`
  - `POST /api/v1/house/public_detail`
- 小程序用户资料、收藏、足迹：见 [api/miniapp-api.md](./api/miniapp-api.md)。
- 发房端第一阶段 HMD 接口：见 [api/publish.md](./api/publish.md)。

待接入：

- 小程序看房计划、通知中心：见 [workstreams/backend-go/active/miniapp-followups.md](./workstreams/backend-go/active/miniapp-followups.md)。
- 管理端 API：尚未开始，后续单独设计。

## 目录怎么用

- [api/](./api/index.md)：接口契约。前端和后端都以这里为准。
- [schema/](./schema/index.md)：数据库字段、枚举、索引。
- [backend/](./backend/index.md)：Go 后端实现边界。
- [flows/](./flows/index.md)：跨模块链路，例如发房到 HPD、小程序找房。
- [workstreams/](./workstreams/index.md)：各端当前推进状态、任务计划和 review report。
- [agents/](./agents/index.md)：不同 Agent 接手任务前的工作规则。
- [product/](./product/index.md)、[frontend/](./frontend/index.md)、[overview/](./overview/index.md)：产品和前端背景，需要时再读。

## 规则

- 不维护统一大而全的 OpenAPI 文件。
- 不维护历史 endpoint map。
- 已废弃接口、旧字段、旧路径说明应删除，不长期堆在 workstreams 里。
- 如果文档之间冲突，以 `api/`、`schema/`、`backend/`、`frontend/` 等规范类文档为准；`workstreams/` 只记录当前推进状态和入口。
