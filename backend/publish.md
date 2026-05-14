# Backend Publish

## 架构

```text
handler/v1/publish
  -> service/publish/PublishService
    -> domain/hmd.Service 写 HMD，并返回 HmdMutationResult
    -> domain/hpd.Service.Apply(changes)
    -> repository/hmd
```

## 当前阶段

- HMD domain service 已拆到 `internal/domain/hmd`。
- HPD domain service 已拆到 `internal/domain/hpd`。
- PublishService 是 handler 的唯一入口。
- `internal/service/publish` 顶层只保留 应用服务 和对外输入类型别名，不承载 HMD 具体实现。
- HMD 写操作返回 `Entity + Changes`。
- PublishService 统一把 changes 交给 `domain/hpd.Service.Apply`。
- HPD 已接入第一期小程序 projector，后续 outbox worker 复用同一套投影逻辑。
- Publish 第一阶段 HMD 链路已完成真实服务联调，route / middleware / Redis session / handler / service / Mongo 落库均已验证通过。

## Handler 层

Publish HTTP 入口放在独立子包：

```text
internal/handler/v1/publish/
  handler.go
  routes.go
  request_common.go
  request_centralized.go
  request_building.go
  request_room_type.go
  request_room.go
  request_decentralized.go
  response.go
```

约束：

- `internal/handler/v1` 根包只保留通用 v1 handler，不继续堆 publish 业务文件。
- handler request 只表达 HTTP 输入，不复用 service DTO 作为请求字段。
- handler 显式把 HTTP request 转成 `service/publish` input。
- handler 不直接调用 HMD domain service 或 repository。
- publish handler 不做 service 错误字符串归类；参数错误、资源不存在、重复资源、数据库错误等语义由 service 返回 `errcode`。

## Service 层

```text
internal/service/publish/
  service.go
  types.go
  dependencies.go
  centralized_project.go
  building.go
  room_type.go
  centralized_room.go
  decentralized_community.go
  decentralized_room.go

internal/domain/hmd/
  hmd_service.go
  hmd_*.go
  dto_*.go
  errors.go
  mapper.go

internal/domain/hpd/
  service.go
  miniapp_projector.go
  miniapp_mapper.go
  miniapp_fanout.go
  errors.go
```

职责：

- `publish/service.go`：只做 应用服务 编排，handler 只依赖这一层。
- `publish/types.go`：保留 应用服务 对外输入类型，避免 handler 直接依赖 HMD 子包。
- `publish/{object}.go`：按业务对象放 publish action，避免 `service.go` 变成透传总线。
- `publish/dependencies.go`：放 publish 入口需要的窄接口。
- `domain/hmd`：负责 HMD 主数据写入、依赖校验、重复校验、mutation changes 和 service 错误语义。
- `domain/hpd`：负责展示层同步入口和 HPD projector，不放在 publish 目录下。

错误约束：

- service 层返回 `errcode.InvalidParam / NotFound / AlreadyExists / DatabaseError` 等明确语义。
- handler 只调用 `response.Err(c, err)`，不通过字符串判断错误类型。

### 数据作用域

PublishService 必须接收或从 context 读取结构化 session principal，并在应用服务层执行数据作用域校验。handler 只负责把 HTTP 输入转成 service input，不在 handler 内拼权限条件；前端也不能通过请求字段决定权限。

principal 结构以 [auth.md](./auth.md) 为准，关键字段为：

```text
principal_type: user | staff
principal_id: ObjectID hex
terminal: miniapp | publish | admin
phone: string
role_codes: string[]
permission_codes: string[]
```

publish 入口约束：

- 所有 publish 业务路由必须使用 `terminal=publish` 的 session principal。
- 全局权限由 `role_codes` 包含 `super_admin` 或 `permission_codes` 包含 `house.manage` 表达。
- 非全局 staff 只能访问 `hs_hpd_entrust_relation` 中 `maintainer_staff_id` 或 `service_staff_id` 等于当前 `principal_id` 且 `relation_status=1` 的房源。
- user 类型 principal 只能访问 `owner_phone` 等于当前 `phone` 且 `relation_status=1` 的房源。
- HMD 不增加 owner/user/staff 字段；数据作用域从 HPD listing 与 entrust relation 起算，再聚合回 HMD 项目、小区、楼栋、房型、房间。
- 列表接口先套数据作用域，再应用 city/district/name 等业务筛选。
- 详情、更新、房态等接口必须先证明当前 principal 可访问目标资源；无权访问时返回权限/不存在语义，不能依赖前端隐藏入口。
- 前端请求中的 `user_id`、`staff_id`、owner、`owner_phone`、`maintainer_staff_id`、`service_staff_id` 等身份过滤字段一律不作为 publish 权限条件。

## HMD Mutation Result

HMD 写操作不再只返回实体，而是返回：

```text
HmdMutationResult
  - Entity
  - Changes
```

`Changes` 描述：

- HMD 动作：created / updated / status_updated
- HMD 实体类型
- HMD 实体 ID
- 影响范围，例如 project、building、room、community

这让后续 HPD projector 可以根据 change 重新读取 HMD 源数据并生成 HPD，而不是在 HMD CRUD 里直接调用 HPD。

## 测试策略

publish 当前按分层验证：

1. `internal/domain/hmd`：Mongo 集成测试，验证 HMD 子能力的数据写入、查询、列表、更新、状态更新、唯一性、归属关系和字段约束。
2. `internal/service/publish`：应用服务 单元测试，验证 HMD 写成功后会调用 HPD `Apply(changes)`，HMD 失败时不调用，读操作不调用，Apply 错误会向上返回。
3. `internal/handler/v1/publish`：已补基础 HTTP binding 测试，重点覆盖 JSON 绑定、ObjectID 解析、参数错误和 service errcode 响应。
4. 真实服务 curl 联调：已验证 route、middleware、wire、config 和依赖装配。

HMD Mongo 集成测试默认跳过，运行方式：

```bash
PUBLISH_HMD_INTEGRATION=1 \
PUBLISH_HMD_TEST_DB=rent-house \
PUBLISH_HMD_TEST_AUTH_SOURCE=rent-house \
PUBLISH_HMD_TEST_ALLOW_RENT_HOUSE=1 \
go test ./internal/domain/hmd -run Integration -count=1 -v
```

说明：

- 正常 `go test ./...` 不会访问 Mongo。
- 集成测试只清理 `itest_` 前缀测试数据。
- 如果有独立测试库，优先改用 `PUBLISH_HMD_TEST_DB=house_manager_test`，不要依赖业务库。

真实服务联调已覆盖：

- `auth/session` 鉴权中间件
- `POST /api/v1/centralized_project/create`
- `POST /api/v1/centralized_project/detail`
- `POST /api/v1/centralized_project/list`
- `POST /api/v1/building/create`
- `POST /api/v1/building/update`
- `POST /api/v1/building/list_by_project`
- `POST /api/v1/room_type/create`
- `POST /api/v1/room_type/list_by_building`
- `POST /api/v1/centralized_room/create`
- `POST /api/v1/centralized_room/update_status`
- `POST /api/v1/centralized_room/list_by_building`
- `POST /api/v1/decentralized_community/create`
- `POST /api/v1/decentralized_community/list`
- `POST /api/v1/decentralized_room/create`
- `POST /api/v1/decentralized_room/update_status`
- `POST /api/v1/decentralized_room/list_by_community`

联调确认：

- 受保护 publish 路由必须携带 `Authorization: Bearer <session-token>`。
- HMD 六个 collection 均可真实落库。
- 分散式房间不写入 `room_type_id`。

## 约束

- 发房系统不是 HMD CRUD 直通层。
- handler 不直接调用 HMD domain service 或 repository。
- `domain/hmd.Service` 不直接依赖 HPD。
- PublishService 只负责统一派发 changes，不写具体投影逻辑。
- 分散式房间当前没有房型模型，不使用 `room_type_id`。
