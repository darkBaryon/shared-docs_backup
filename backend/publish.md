# Backend Publish

## 架构

```text
handler/v1/publish
  -> service/publish/PublishService
    -> service/publish/hmd.Service 写 HMD，并返回 HmdMutationResult
    -> service/publish/hpd.Service.Apply(changes)
    -> repository/hmd
```

## 当前阶段

- HMD 子 service 已拆到 `internal/service/publish/hmd`。
- HPD 子 service 已拆到 `internal/service/publish/hpd`。
- PublishService 是 handler 的唯一入口。
- `internal/service/publish` 顶层只保留 facade 和对外输入类型别名，不承载 HMD 具体实现。
- HMD 写操作返回 `Entity + Changes`。
- PublishService 统一把 changes 交给 `hpd.Service.Apply`。
- HPD 当前 `Apply` 是 no-op 预留点，后续接入 projector 或 outbox worker 复用的投影逻辑。

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
- handler 不直接调用 HMD 子 service 或 repository。
- publish handler 不做 service 错误字符串归类；参数错误、资源不存在、重复资源、数据库错误等语义由 service 返回 `errcode`。

## Service 层

```text
internal/service/publish/
  service.go
  types.go
  hmd/
    hmd_service.go
    hmd_*.go
    dto_*.go
    errors.go
    mapper.go
  hpd/
    service.go
```

职责：

- `publish/service.go`：只做 facade 编排，handler 只依赖这一层。
- `publish/types.go`：保留 facade 对外输入类型，避免 handler 直接依赖 HMD 子包。
- `publish/hmd`：负责 HMD 主数据写入、依赖校验、重复校验、mutation changes 和 service 错误语义。
- `publish/hpd`：负责展示层同步入口，当前为 no-op。

错误约束：

- service 层返回 `errcode.InvalidParam / NotFound / AlreadyExists / DatabaseError` 等明确语义。
- handler 只调用 `response.Err(c, err)`，不通过字符串判断错误类型。

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

1. `internal/service/publish/hmd`：Mongo 集成测试，验证 HMD 子能力的数据写入、查询、列表、更新、状态更新、唯一性、归属关系和字段约束。
2. `internal/service/publish`：facade 单元测试，验证 HMD 写成功后会调用 HPD `Apply(changes)`，HMD 失败时不调用，读操作不调用，Apply 错误会向上返回。
3. `internal/handler/v1/publish`：已补基础 HTTP binding 测试，重点覆盖 JSON 绑定、ObjectID 解析、参数错误和 service errcode 响应。
4. 真实服务 curl 联调：最后验证 route、middleware、wire、config 和依赖装配。

HMD Mongo 集成测试默认跳过，运行方式：

```bash
PUBLISH_HMD_INTEGRATION=1 \
PUBLISH_HMD_TEST_DB=rent-house \
PUBLISH_HMD_TEST_AUTH_SOURCE=rent-house \
PUBLISH_HMD_TEST_ALLOW_RENT_HOUSE=1 \
go test ./internal/service/publish/hmd -run Integration -count=1 -v
```

说明：

- 正常 `go test ./...` 不会访问 Mongo。
- 集成测试只清理 `itest_` 前缀测试数据。
- 如果有独立测试库，优先改用 `PUBLISH_HMD_TEST_DB=house_manager_test`，不要依赖业务库。

## 约束

- 发房系统不是 HMD CRUD 直通层。
- handler 不直接调用 HMD 子 service 或 repository。
- `publish/hmd.Service` 不直接依赖 HPD。
- PublishService 只负责统一派发 changes，不写具体投影逻辑。
- 分散式房间当前没有房型模型，不使用 `room_type_id`。
