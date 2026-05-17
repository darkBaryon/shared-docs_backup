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
- Admin Auth 第一阶段：`admin_auth/login`、`admin_auth/session`、`admin_auth/logout` 已实现，员工手机号密码登录、角色权限回填、Redis session 恢复与退出已接通。
- Admin Staff 第一阶段：`staff/create`、`staff/list`、`staff/detail`、`staff/update`、`staff/disable` 已实现，支持创建员工主体、密码认证、角色关系、员工列表分页查询、员工详情查看、员工资料维护、角色替换、员工禁用和 `staff.edit/staff.view` 权限校验。
- Admin Role 第一阶段：`role/list`、`role/detail`、`role/create`、`role/update` 已实现，支持角色分页查询、详情查看、角色创建、角色编辑、权限关系维护、权限变更后员工 admin token 失效和 `role.view/role.edit` 权限校验。
- Admin Provider 第一阶段：`provider/list`、`provider/detail`、`provider/create`、`provider/update`、`provider/disable` 已实现，支持发房方账号列表、详情、创建、手机号修改、禁用、非事务创建补偿、publish token 失效和 `provider.view/provider.edit` 权限校验；当前阶段不写 `hs_lld_profile`。
- 基础 CI：GitHub Actions 已接入 `gofmt -l .`、`go test ./...`、`go build ./...`，真实 Mongo / Redis 集成测试仍默认跳过。
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
