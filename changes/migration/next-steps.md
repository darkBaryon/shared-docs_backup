# 接下来要做的事情

这份文档只回答两件事：

1. 接下来要实现什么需求，准备怎么实现
2. 代码准备怎么组织，文件怎么拆，每个文件负责什么

前提：

> 当前 `internal/service/publish` 里的代码还没有完成 review，不能当成已完成实现，只能当成草稿。

---

## 1. 要实现的需求

### 1.1 当前阶段要做的需求

当前阶段不是直接把整个发房系统做完，而是先做发房系统第一期。

第一期目标：

- 先把 `publish` 域的整体架构定清楚
- 先把 HMD 相关的发房能力接成可调用 API
- 给后续 HPD 接入留出正确位置

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

当前最先要做的不是 handler，也不是 HPD，而是先整理 `internal/service/publish`。

原因：

- 现在的 `hmd_service.go` 已经太大
- 现在的 `dto.go` 已经没有边界

如果不先整理文件结构，后面无论继续写：

- publish 上层 service
- handler
- HPD

都会继续堆到错误的文件里。

### 2.3 这一步的实施顺序

接下来正确顺序是：

1. 先 review 当前 `internal/service/publish` 草稿
2. 先重组 `internal/service/publish` 的文件结构
3. 再实现 publish 上层 service
4. 再实现 publish handler 和路由
5. 再继续接 HPD

---

## 3. 代码实现方式

## 3.1 总体代码架构

推荐分层如下：

```text
router
  -> handler/v1/publish
    -> service/publish/service.go
      -> service/publish/hmd_*.go
      -> service/publish/hpd_*.go
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
- `HMD 子 service`
  - 负责 HMD 相关校验和组合调用
- `HPD 子 service`
  - 负责后续发布和展示层数据
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

当前要先整理成下面这套结构：

```text
internal/service/publish/
  service.go
  hmd_service.go
  hmd_centralized_project.go
  hmd_building.go
  hmd_room_type.go
  hmd_centralized_room.go
  hmd_decentralized_community.go
  hmd_decentralized_room.go
  hmd_validation.go
  mapper.go
  dto_common.go
  dto_centralized.go
  dto_room_type.go
  dto_decentralized.go
  dto_room.go
  hpd_service.go
```

### `service.go`

功能：

- publish 域上层 service
- 作为 publish 域最终业务入口
- 被 handler 直接调用

负责：

- 定义 `PublishService`
- 注入 HMD 子 service
- 后续注入 HPD 子 service
- 暴露 publish action 对应的方法

不负责：

- 直接写 repository 细节
- 承载所有 HMD 具体实现

### `hmd_service.go`

功能：

- HMD 子 service 的聚合入口

负责：

- 定义 `HmdService`
- 持有所有 HMD repository
- 只保留构造函数和聚合结构

不负责：

- 承载所有对象方法
- 承载全部 helper

### `hmd_centralized_project.go`

功能：

- 集中式项目相关方法

负责：

- `CreateCentralizedProject`
- `GetCentralizedProject`
- `ListCentralizedProjects`
- `UpdateCentralizedProject`

### `hmd_building.go`

功能：

- 楼栋相关方法

负责：

- `CreateBuilding`
- `GetBuilding`
- `ListBuildingsByProject`
- `UpdateBuilding`

### `hmd_room_type.go`

功能：

- 集中式房型相关方法

负责：

- `CreateRoomType`
- `GetRoomType`
- `ListRoomTypesByProject`
- `ListRoomTypesByBuilding`
- `UpdateRoomType`

### `hmd_centralized_room.go`

功能：

- 集中式房间相关方法

负责：

- `CreateCentralizedRoom`
- `GetCentralizedRoom`
- `ListCentralizedRoomsByProject`
- `ListCentralizedRoomsByBuilding`
- `UpdateCentralizedRoom`
- `UpdateCentralizedRoomStatus`

### `hmd_decentralized_community.go`

功能：

- 分散式小区主档相关方法

负责：

- `CreateDecentralizedCommunity`
- `GetDecentralizedCommunity`
- `ListDecentralizedCommunities`
- `UpdateDecentralizedCommunity`

### `hmd_decentralized_room.go`

功能：

- 分散式房间相关方法

负责：

- `CreateDecentralizedRoom`
- `GetDecentralizedRoom`
- `ListDecentralizedRoomsByCommunity`
- `UpdateDecentralizedRoom`
- `UpdateDecentralizedRoomStatus`

### `hmd_validation.go`

功能：

- 放 HMD 子域的校验 helper

负责：

- 项目存在校验
- 楼栋归属校验
- 房型归属校验
- 房号重复校验
- 小区重复校验
- 其他 HMD 依赖关系检查

### `mapper.go`

功能：

- 放 DTO / model 转换 helper

负责：

- `GeoPointInput -> model.GeoPoint`
- `TaggedImageInput -> model.TaggedImage`
- slice clone / trim
- 公共字段转换

### `dto_common.go`

功能：

- 放 DTO 通用结构

负责：

- `GeoPointInput`
- `TaggedImageInput`
- 其他跨对象复用的小结构

### `dto_centralized.go`

功能：

- 放集中式项目和楼栋相关 DTO

负责：

- `ListCentralizedProjectsInput`
- `CreateCentralizedProjectInput`
- `UpdateCentralizedProjectInput`
- `CreateBuildingInput`
- `UpdateBuildingInput`

### `dto_room_type.go`

功能：

- 放房型相关 DTO

负责：

- `CreateRoomTypeInput`
- `UpdateRoomTypeInput`

### `dto_decentralized.go`

功能：

- 放分散式小区主档相关 DTO

负责：

- `ListDecentralizedCommunitiesInput`
- `CreateDecentralizedCommunityInput`
- `UpdateDecentralizedCommunityInput`

### `dto_room.go`

功能：

- 放房间相关 DTO

负责：

- `CreateCentralizedRoomInput`
- `UpdateCentralizedRoomInput`
- `UpdateCentralizedRoomStatusInput`
- `CreateDecentralizedRoomInput`
- `UpdateDecentralizedRoomInput`
- `UpdateDecentralizedRoomStatusInput`

### `hpd_service.go`

功能：

- 预留 HPD 子 service 落点

当前阶段：

- 可以先建空文件或最小骨架
- 暂不实现具体逻辑

目的：

- 明确 publish 后续一定会接入 HPD

---

## 5. handler 层文件架构

在 `service/publish` 整理完成后，再新增：

```text
internal/handler/v1/publish/
  handler.go
  request.go
  response.go
```

### `handler.go`

功能：

- 实现 publish action 对应的 handler method

负责：

- 绑定请求
- 调用 `PublishService`
- 返回统一响应

### `request.go`

功能：

- 放 handler 层请求结构

说明：

- 这层 request 不要和 service DTO 混在一起
- handler request 只服务 HTTP 输入

### `response.go`

功能：

- 放 handler 层返回结构
- 或者放 response mapping helper

---

## 6. 接下来先做什么

下一步不要继续往 `hmd_service.go` 和 `dto.go` 里堆代码。

下一步要先做的是：

1. 重组 `internal/service/publish` 文件结构
2. 把当前 `hmd_service.go` 拆开
3. 把当前 `dto.go` 拆开
4. 再开始实现 `service.go`

只有这一步做完，后面的：

- publish 上层 service
- publish handler
- HPD 子 service

才值得继续写。
