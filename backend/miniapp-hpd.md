# Miniapp HPD Plan

本文档定义“小程序展示层 HPD”的第一期实现规划。

目标不是一次做完所有 HPD，而是先支撑小程序前端已经具备的找房、列表、详情链路。

## 1. 功能范围

### 1.1 第一期要做

第一期只做小程序端读取所需的展示层数据。

核心能力：

- 从 HMD 生成小程序可读的 HPD 展示快照
- 支持小程序房源列表查询
- 支持小程序房源详情查询
- 支持按城市、区域、价格、租住方式等基础条件筛选
- 支持只展示在线房源
- 支持 HMD 变更后刷新对应 HPD 快照

第一期发布源：

- 集中式房间：`hs_hmd_room_centralized`
- 分散式房间：`hs_hmd_room_decentralized`

第一期不把集中式房型模板作为小程序发布源。

原因：

- 小程序最终展示的是可租房源实例。
- `hs_hmd_room_type_centralized` 当前没有 `rent_mode`，无法稳定生成小程序必需的租住方式。
- 如果后续要“按房型发布”，需要先补业务规则和模型字段，不能临时复用集中式房间逻辑。

### 1.2 第一期不做

- 后台管理 HPD
- 房东端 HPD
- 审核流
- 上下架完整流程
- outbox worker
- 搜索引擎或复杂推荐排序
- 从 HMD 反查小程序展示字段
- 录入即发布

## 2. 数据模型口径

第一期使用两个核心集合：

- `hs_hpd_listing`
- `hs_hpd_miniapp_listing`

其中：

- `hs_hpd_listing` 是发布主实体，只保存统一 listing identity、来源和发布状态。
- `hs_hpd_miniapp_listing` 是小程序 read model，保存列表和详情需要的冗余快照。

小程序接口原则上只读 `hs_hpd_miniapp_listing`。

需要注意：

- HPD 不是 HMD 的一层 view。
- HMD 是主数据，HPD 是展示层快照。
- 小程序不能直接依赖 HMD collection。
- 小程序详情第一期也优先读 `hs_hpd_miniapp_listing`，不要运行时拼 HMD。
- 后台管理和发房端/房东端后续单独设计 read model，不复用小程序展示表。

## 3. 同步策略

### 3.1 当前阶段

当前 `PublishService` 已经在 HMD 写操作成功后调用：

```text
hpd.Service.Apply(changes)
```

第一期 HPD 要把这个 no-op 改成真实 projector 调用。

同步方式：

```text
PublishService
  -> hmd.Service 写 HMD
  -> HmdMutationResult{Entity, Changes}
  -> hpd.Service.Apply(changes)
    -> MiniappProjector
      -> 读取 HMD 当前源数据
      -> 生成 / 更新 hs_hpd_listing
      -> 生成 / 更新 hs_hpd_miniapp_listing
```

第一期先做同步写。

也就是说：

- HMD 写成功后立即刷新 HPD。
- 如果 HPD 刷新失败，接口先返回错误。
- 后续如果需要更强可靠性，再升级事务内 outbox + 异步 worker。
- projector 不自动把新房源置为已上架。
- 第一阶段联调如果需要小程序可查数据，通过 seed 或测试 fixture 写入 `listing_status=3`。

### 3.2 后续 outbox

outbox 只改变投递方式，不改变 projector 逻辑。

后续形态：

```text
HMD 写事务
  -> 写 HMD
  -> 写 outbox event

Worker
  -> 读取 outbox event
  -> hpd.Service.Apply(changes)
```

## 4. Projector 刷新规则

### 4.1 来源对象变化

| HMD change scope | 刷新策略 |
| --- | --- |
| `centralized_room` | 刷新该集中式房间对应 listing / miniapp_listing |
| `decentralized_room` | 刷新该分散式房间对应 listing / miniapp_listing |

### 4.2 父级对象变化

父级主数据变化会影响多个小程序展示快照。

| HMD change scope | 需要刷新 |
| --- | --- |
| `centralized_project` | 该项目下所有集中式房间 miniapp_listing |
| `building` | 该楼栋下所有集中式房间 miniapp_listing |
| `room_type_centralized` | 引用该房型的集中式房间 miniapp_listing |
| `decentralized_community` | 该小区下所有分散式房间 miniapp_listing |

第一期可以先用 Mongo 查询 fan-out，不引入异步队列。

## 5. 字段生成规则

### 5.1 集中式房间

来源：

- 房间：`hs_hmd_room_centralized`
- 楼栋：`hs_hmd_building`
- 项目：`hs_hmd_centralized`
- 房型：`hs_hmd_room_type_centralized`，可选

生成字段：

- `source_type`: `centralized_room`
- `source_id`: 房间 `_id`
- `asset_mode`: `centralized`
- `rent_mode`: 房间 `rent_mode`
- `city` / `district` / `address_text` / `geo`: 项目字段
- `community_name`: 项目名
- `building_or_community_name`: 楼栋名或项目名
- `title`: 项目名 + 楼栋名 + 房间号
- `subtitle`: 户型、面积、朝向、楼层组合
- `price`: 房间 `rent`
- `layout_text`: 房间 `layout_text`，为空时可由房型房厅卫字段组装
- `images`: 房间图片优先，房型图片兜底
- `listing_facilities`: 房间配置 + 楼栋/项目展示配置
- `is_online`: `listing_status=3` 且房间 `room_status=1` 时为 `1`

### 5.2 分散式房间

来源：

- 房间：`hs_hmd_room_decentralized`
- 小区：`hs_hmd_decentralized`

生成字段：

- `source_type`: `decentralized_room`
- `source_id`: 房间 `_id`
- `asset_mode`: `decentralized`
- `rent_mode`: 房间 `rent_mode`
- `city` / `district` / `biz_area` / `address_text` / `geo` / `subway_station`: 小区字段
- `community_name`: 小区名
- `building_or_community_name`: 小区名
- `title`: 小区名 + 房间号
- `subtitle`: 户型、面积、朝向、楼层组合
- `price`: 房间 `rent`
- `layout_text`: 房间 `layout_text`
- `images`: 房间图片
- `listing_facilities`: 房间配置
- `is_online`: `listing_status=3` 且房间 `room_status=1` 时为 `1`

### 5.3 发布状态与房态关系

第一阶段遵守真实业务边界：

- `listing_status` 是发布状态，不由 HMD `room_status` 自动推导。
- HMD `room_status` 是房间状态，只作为小程序可见性的安全门。
- projector 不负责“自动上架”。
- 小程序展示条件是 `listing_status=3` 且 HMD `room_status=1`。

映射表：

| `listing_status` | HMD `room_status` | `is_online` | 说明 |
| --- | --- | --- | --- |
| `3` 已上架 | `1` 未出租/可租 | `1` | 小程序可展示 |
| `3` 已上架 | 非 `1` | `0` | 已发布但房间不可租，不展示 |
| 非 `3` | 任意 | `0` | 未发布、下架、已出租、删除等不展示 |

测试说明：

- 第一阶段没有发布/审核接口时，不为了测试把 projector 改成录入即发布。
- 需要小程序 E2E 数据时，通过 seed 或 fixture 创建 `listing_status=3` 的 `hs_hpd_listing`。
- 后续审核/上下架流程接入后，由对应发布流程维护 `listing_status`、`published_at`、`offline_at`。

## 6. API 规划

第一期先支撑小程序找房和详情。

确认接口：

```text
POST /api/v1/house/search
POST /api/v1/house/public_detail
```

原则：

- 不为了旧 Python 接口做兼容。
- 当前小程序前端已经确认使用 `house` 模块，因此 Go 后端按 `house/search`、`house/public_detail` 接入。
- API 路径必须符合 `POST /api/v{version}/{模块}/{动作}`。
- 小程序是前端端类型，不作为这里的 API 模块名。
- 禁止使用 `/api/v1/miniapp/listing/search` 这类多级业务路径。

## 7. 代码架构

### 7.1 Model

```text
internal/model/
  hpd.go
  hpd_enum.go
  hpd_validation.go
```

职责：

- 定义 `HpdListing`
- 定义 `HpdMiniappListing`
- 定义 HPD 枚举
- 定义 create / update 字段校验

### 7.2 Repository

```text
internal/repository/hpd/
  listing_repository.go
  miniapp_listing_repository.go
  fields.go
  indexes.go 或 scripts/mongo_hpd_indexes.js
```

职责：

- `listing_repository.go`
  - `Create`
  - `FindByID`
  - `FindBySource`
  - `UpsertBySource`
  - `UpdateStatus`
- `miniapp_listing_repository.go`
  - `FindByListingID`
  - `UpsertByListingID`
  - `SearchMiniapp`
  - `FindOnlineDetail`
- `fields.go`
  - 字段白名单
  - filter helper
- `indexes.go` 或脚本
  - `source_type + source_id` 唯一索引
  - `listing_id` 唯一索引
  - 小程序筛选索引

### 7.3 Service

```text
internal/service/hpd/
  service.go
  miniapp_projector.go
  miniapp_mapper.go
  miniapp_fanout.go
  errors.go
```

职责：

- `service.go`
  - 保留 `Apply(ctx, changes)` 入口
  - 不直接写复杂映射逻辑
- `miniapp_projector.go`
  - 根据 change 调用具体刷新方法
  - 刷新 listing 和 miniapp_listing
- `miniapp_mapper.go`
  - HMD -> HPD 字段映射
  - 标题、价格、户型、图片、在线状态生成
- `miniapp_fanout.go`
  - 项目、楼栋、房型、小区变更时查找受影响房间
- `errors.go`
  - 返回明确 `errcode`

小程序读接口 service 建议单独放：

```text
internal/service/miniapp/
  listing_service.go
  dto.go
```

原因：

- `hpd` 负责投影写入。
- `miniapp` service 负责小程序读查询。
- 不把小程序读逻辑塞进 publish service。

### 7.4 Handler

```text
internal/handler/v1/miniapp/
  handler.go
  routes.go
  house.go
  requests.go
```

职责：

- 绑定小程序查询参数
- 调用 `service/miniapp`
- 返回小程序展示结构
- 注册 `POST /api/v1/house/search`
- 注册 `POST /api/v1/house/public_detail`

handler 不直接读 HMD，也不直接调 projector。

## 8. 索引规划

### `hs_hpd_listing`

建议索引：

```text
source_type_1_source_id_1 unique
listing_status_1_updated_at_-1
```

### `hs_hpd_miniapp_listing`

建议索引：

```text
listing_id_1 unique
source_type_1_source_id_1
is_online_1_city_1_district_1_price_1
is_online_1_rent_mode_1_price_1
is_online_1_weight_score_-1_updated_at_-1
```

## 9. 测试计划

### Repository

- `FindBySource`
- `UpsertBySource`
- `UpsertByListingID`
- `SearchMiniapp`
- `FindOnlineDetail`

### Projector

- 集中式房间生成 listing / miniapp_listing
- 分散式房间生成 listing / miniapp_listing
- 项目变更 fan-out 刷新集中式房间
- 楼栋变更 fan-out 刷新集中式房间
- 房型变更 fan-out 刷新引用房间
- 小区变更 fan-out 刷新分散式房间
- 分散式房间不依赖房型

### Handler / API

- 搜索参数绑定
- 详情 ObjectID 校验
- 只返回在线房源
- 无结果返回空列表

### E2E

- 发房端创建 HMD
- projector 生成 HPD
- 小程序查询列表命中
- 小程序详情返回展示字段

## 10. 实施顺序

1. 确认当前小程序前端实际调用路径和字段。
2. 定稿小程序查询 API。
3. 补 `model/hpd*`。
4. 补 `repository/hpd`。
5. 补 HPD 索引脚本或初始化逻辑。
6. 实现 `hpd.Service.Apply(changes)` 的 miniapp projector。
7. 实现 `service/miniapp` 和 `handler/v1/miniapp`。
8. 跑 repository / projector / handler 测试。
9. 用发房端创建 HMD，再用小程序接口查询 HPD 做 E2E。

## 11. 关键约束

- 小程序只读 HPD，不读 HMD。
- `hmd.Service` 不直接依赖 HPD。
- projector 根据 HMD 当前源数据重建 HPD，不接收 handler 拼好的展示字段。
- 第一阶段不做集中式房型发布源。
- 第一阶段不做 outbox，先保证同步 projector 跑通。
- 第一阶段不做录入即发布，测试数据用 seed 灌入已上架状态。
- 不兼容旧数据库结构；字段以当前 schema 和当前前端确认需求为准。
