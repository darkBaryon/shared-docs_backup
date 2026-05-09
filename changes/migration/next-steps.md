# 接下来要做的事情

这份文档只回答两件事：

1. 接下来要实现什么需求，准备怎么实现
2. 代码准备怎么组织，文件怎么拆，每个文件负责什么

前提：

> 当前 publish 第一版 HMD 后端链路已经完成真实服务联调。下一步不是继续证明后端能否跑通，而是让发房前端接入这组接口，并继续推进 HPD。

补充：

> HPD 先做小程序展示层，因为小程序前端已有代码，可以直接进入 E2E。

---

## 1. 要实现的需求

### 1.1 当前阶段要做的需求

当前阶段不是直接把整个发房系统做完，而是把发房系统第一期从“后端已验证”推进到“前端可 E2E”。

第一期目标：

- 发房前端接入 HMD 相关发房 API
- 用真实服务完成前后端 E2E
- 根据前端联调反馈补齐必要的 handler case 和文档
- 给后续 HPD 接入留出正确位置

HPD 当前目标：

- 先实现小程序列表和详情需要的 HPD
- 从集中式房间和分散式房间生成展示层快照
- 不先做后台管理 HPD、房东端 HPD、审核流和 outbox worker

这一期要覆盖的业务对象：

- 集中式项目
- 楼栋
- 集中式房型
- 集中式房间
- 分散式小区主档
- 分散式房间

这一期要覆盖的动作：

- 创建
- 详情
- 列表
- 更新
- 房态更新

对应 API 动作仍按已定口径：

```text
POST /api/v1/publish/{action}
```

例如：

- `create_centralized_project`
- `list_buildings_by_project`
- `create_decentralized_room`
- `update_centralized_room_status`

### 1.2 这期不做完的需求

这一期先不把 HPD 完整做完，也不先做后台管理系统。

但必须在架构上明确：

- `publish` 不等于 HMD CRUD
- 后续录房动作会继续接入 HPD
- 后续还会产出：
  - 房东端展示数据
  - 小程序展示数据
  - 后台展示数据

---

## 2. 实现思路

### 2.1 总体思路

`publish` 是业务域，不是数据域。

所以实现时不能走这种结构：

```text
handler -> repository/hmd
```

也不能把 `publish` 写成：

```text
一组 HMD 表的 CRUD 接口
```

正确思路是：

```text
handler
  -> publish 上层 service
    -> HMD 子 service
    -> HPD 子 service（后续）
    -> repository
```

### 2.2 先做什么

当前最先要做的是发房前端 E2E。

原因：

- service、handler、route、Wire 已经接入
- HMD 写操作已经返回 `Entity + Changes`
- PublishService 已经统一派发 `hpd.Service.Apply(changes)`
- 真实服务 curl 联调已经通过
- handler request binding 已有基础测试覆盖
- HPD 仍是 no-op 预留点

### 2.3 这一步的实施顺序

接下来正确顺序是：

1. 发房前端按 `api/publish.md` 接入 HMD 录入、详情、列表、更新和房态流转
2. 用真实服务做前后端 E2E
3. 修复 E2E 暴露出的字段、校验、响应结构和交互问题
4. 随接口扩展继续补 handler / service 边界用例
5. 按 [../../backend/miniapp-hpd.md](../../backend/miniapp-hpd.md) 设计并实现小程序 HPD projector

---

## 3. 代码实现方式

## 3.1 总体代码架构

推荐分层如下：

```text
router
  -> handler/v1/publish
    -> service/publish/service.go
      -> service/publish/hmd/*.go
      -> service/publish/hpd/*.go
      -> repository/hmd
      -> repository/hpd
```

职责划分：

- `router`
  - 只做路径注册
- `handler`
  - 只做 HTTP 参数绑定、调用 service、返回 response
- `publish 上层 service`
  - 负责 publish 业务动作语义
  - 统一处理 HMD mutation result 中的 changes
  - 只做 facade 编排，不承载 HMD 具体实现
- `HMD 子 service`
  - 负责 HMD 相关校验和组合调用
  - 写操作返回 `HmdMutationResult`
  - 返回明确 `errcode`，不把错误语义留给 handler 猜
- `HPD 子 service`
  - 当前 no-op
  - 后续通过 `Apply(changes)` 调 projector 或 outbox worker 复用的投影逻辑
- `repository`
  - 负责数据库读写

## 3.2 repository 层

当前先继续复用已有结构：

```text
internal/repository/common
internal/repository/auth
internal/repository/hmd
```

后续新增：

```text
internal/repository/hpd
```

Repository 的职责不变：

- 查库
- 写库
- 字段边界
- 软删和状态约束

Repository 不负责：

- publish 业务语义
- 展示层编排

---

## 4. 文件架构

## 4.1 `internal/service/publish`

当前已整理为下面这套结构：

```text
internal/service/publish/
  service.go
  types.go
  hmd/
    hmd_service.go
    hmd_centralized_project.go
    hmd_building.go
    hmd_room_type.go
    hmd_centralized_room.go
    hmd_decentralized_community.go
    hmd_decentralized_room.go
    hmd_validation.go
    hmd_change.go
    errors.go
    mapper.go
    dto_common.go
    dto_centralized.go
    dto_room_type.go
    dto_decentralized.go
    dto_room.go
  hpd/
    service.go
```

### `service.go`

功能：

- publish 域上层 service
- 作为 publish 域最终业务入口
- 被 handler 直接调用

负责：

- 定义 `PublishService`
- 注入 `publish/hmd.Service`
- 注入 `publish/hpd.Service`
- 暴露 publish action 对应的方法
- 统一派发 HMD changes 给 HPD

不负责：

- 直接写 repository 细节
- 承载所有 HMD 具体实现
- 用字符串判断错误类型

### `types.go`

功能：

- 暴露 publish facade 的对外输入类型

负责：

- 让 handler 只依赖 `internal/service/publish`
- 通过类型别名复用 HMD 子 service 的第一期输入结构

不负责：

- 承载 HMD 实现逻辑

### `hmd/hmd_service.go`

功能：

- HMD 子 service 的聚合入口

负责：

- 定义 `hmd.Service`
- 持有所有 HMD repository
- 只保留构造函数和聚合结构

不负责：

- 承载所有对象方法
- 承载全部 helper

### `hmd/hmd_centralized_project.go`

功能：

- 集中式项目相关方法

负责：

- `CreateCentralizedProject`
- `GetCentralizedProject`
- `ListCentralizedProjects`
- `UpdateCentralizedProject`

### `hmd/hmd_building.go`

功能：

- 楼栋相关方法

负责：

- `CreateBuilding`
- `GetBuilding`
- `ListBuildingsByProject`
- `UpdateBuilding`

### `hmd/hmd_room_type.go`

功能：

- 集中式房型相关方法

负责：

- `CreateRoomType`
- `GetRoomType`
- `ListRoomTypesByProject`
- `ListRoomTypesByBuilding`
- `UpdateRoomType`

### `hmd/hmd_centralized_room.go`

功能：

- 集中式房间相关方法

负责：

- `CreateCentralizedRoom`
- `GetCentralizedRoom`
- `ListCentralizedRoomsByProject`
- `ListCentralizedRoomsByBuilding`
- `UpdateCentralizedRoom`
- `UpdateCentralizedRoomStatus`

### `hmd/hmd_decentralized_community.go`

功能：

- 分散式小区主档相关方法

负责：

- `CreateDecentralizedCommunity`
- `GetDecentralizedCommunity`
- `ListDecentralizedCommunities`
- `UpdateDecentralizedCommunity`

### `hmd/hmd_decentralized_room.go`

功能：

- 分散式房间相关方法

负责：

- `CreateDecentralizedRoom`
- `GetDecentralizedRoom`
- `ListDecentralizedRoomsByCommunity`
- `UpdateDecentralizedRoom`
- `UpdateDecentralizedRoomStatus`

硬约束：

- 这里必须修正当前草稿里的 `room_type_id` 口径问题
- 分散式房间不能继续引用 `hs_hmd_room_type_centralized`
- 当前阶段已经定稿：分散式暂不提供房型模型
- 因此分散式房间当前不提供 `RoomTypeID`
- 后续如果要支持分散式房型，必须单独定义分散式房型模型和数据结构

### `hmd/hmd_validation.go`

功能：

- 放 HMD 子域的校验 helper

负责：

- 项目存在校验
- 楼栋归属校验
- 房型归属校验
- 房号重复校验
- 小区重复校验
- 其他 HMD 依赖关系检查

边界约束：

- 这里只放跨对象共享校验
- 某个对象私有校验，尽量留在对应的 `hmd_*.go` 文件里
- 不允许把 `hmd_validation.go` 继续堆成新的 helper 大杂烩

### `hmd/errors.go`

功能：

- 放 HMD 子 service 的错误语义 helper

负责：

- 返回 `errcode.InvalidParam`
- 返回 `errcode.NotFound`
- 返回 `errcode.AlreadyExists`
- 返回 `errcode.DatabaseError`

约束：

- handler 不再通过字符串归类 service 错误

### `hmd/mapper.go`

功能：

- 放 DTO / model 转换 helper

负责：

- `GeoPointInput -> model.GeoPoint`
- `TaggedImageInput -> model.TaggedImage`
- slice clone / trim
- 公共字段转换

### `hmd/dto_common.go`

功能：

- 放 DTO 通用结构

负责：

- `GeoPointInput`
- `TaggedImageInput`
- 其他跨对象复用的小结构

### `hmd/dto_centralized.go`

功能：

- 放集中式项目和楼栋相关 DTO

负责：

- `ListCentralizedProjectsInput`
- `CreateCentralizedProjectInput`
- `UpdateCentralizedProjectInput`
- `CreateBuildingInput`
- `UpdateBuildingInput`

### `hmd/dto_room_type.go`

功能：

- 放房型相关 DTO

负责：

- `CreateRoomTypeInput`
- `UpdateRoomTypeInput`

### `hmd/dto_decentralized.go`

功能：

- 放分散式小区主档相关 DTO

负责：

- `ListDecentralizedCommunitiesInput`
- `CreateDecentralizedCommunityInput`
- `UpdateDecentralizedCommunityInput`

### `hmd/dto_room.go`

功能：

- 放房间相关 DTO

负责：

- `CreateCentralizedRoomInput`
- `UpdateCentralizedRoomInput`
- `UpdateCentralizedRoomStatusInput`
- `CreateDecentralizedRoomInput`
- `UpdateDecentralizedRoomInput`
- `UpdateDecentralizedRoomStatusInput`

### `hpd/service.go`

功能：

- 预留 HPD 子 service 落点

当前阶段：

- 可以先建最小接口或最小骨架
- 暂不实现具体业务逻辑
- 需要在文件注释中写清楚：当前阶段是 no-op 预留，后续由 `PublishService` 决定调用点

目的：

- 明确 publish 后续一定会接入 HPD

---

## 5. handler 层文件架构

当前 handler 层采用：

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

### `handler.go`

功能：

- 持有 `PublishService`
- 暴露 handler 构造函数

负责：

- 保持 handler 聚合结构清晰

### `routes.go`

功能：

- 注册 publish action 对应路由

### `request_*.go`

功能：

- 放 handler 层请求结构

说明：

- 这层 request 不要和 service DTO 混在一起
- handler request 只服务 HTTP 输入
- request 到 service input 的转换要显式写在 handler 层

### `response.go`

功能：

- 放 publish response helper
- 只调用 `response.Err(c, err)`，错误语义来自 service 返回的 `errcode`

---

## 6. 接下来先做什么

当前 `service/publish` 和 handler 文件边界已经整理完成，不要再把 HMD 细节塞回 publish 顶层。

下一步要先做的是：

1. 发房前端接入 `POST /api/v1/publish/{action}` 第一阶段接口
2. 用真实服务完成前后端 E2E
3. 修复 E2E 暴露出的字段、校验、响应结构和交互问题
4. 继续补充 handler / publish facade / HMD service 的边界用例
5. 按 [../../backend/miniapp-hpd.md](../../backend/miniapp-hpd.md) 实现小程序 HPD projector
