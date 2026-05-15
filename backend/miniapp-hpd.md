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
domain/listingprojection.Service.Apply(changes)
```

当前第一期 HPD 已接入真实 miniapp projector 调用。

同步方式：

```text
PublishService
  -> domain/hmd.Service 写 HMD
  -> HmdMutationResult{Entity, Changes}
  -> domain/listingprojection.Service.Apply(changes)
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
  -> domain/listingprojection.Service.Apply(changes)
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

## 7. 代码架构 Review 结论

旧计划里把小程序读接口放在：

```text
internal/service/miniapp/
internal/handler/v1/miniapp/
```

该方案需要调整。

原因：

- `miniapp` 是前端端类型，不是后端业务模块。
- 对外 API 模块已经确认是 `house`，不是 `miniapp`。
- 后续还会有 `favorite`、`history`、`user` 等小程序端接口，必须继续按 `service/miniapp/{module}` 拆分，不能塞进 `service/miniapp` 单个大包。
- HPD 是内部展示层 read model，不是对外 API 模块；`domain/listingprojection` 只负责 projection / lifecycle / `Apply(changes)`，不承载小程序读 API。

最终依赖链必须是：

```text
handler/v1/miniapp/house
  -> service/miniapp/house
    -> repository/hpd.MiniappListingRepository
      -> hs_hpd_miniapp_listing
```

明确禁止：

```text
handler -> domain/listingprojection projector
handler -> repository/hpd
handler -> hmd
service/miniapp/house -> hmd
```

## 8. 最终目录结构

### 8.1 Model

```text
internal/model/
  hpd/
    model.go
    enum.go
    validation.go
```

职责：

- 定义 `HpdListing`
- 定义 `HpdMiniappListing`
- 定义 HPD 枚举
- 定义 create / update 字段校验

### 8.2 Repository

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
  - `CountMiniapp`
  - `FindOnlineDetail`
- `fields.go`
  - 字段白名单
  - filter helper
- `indexes.go` 或脚本
  - `source_type + source_id` 唯一索引
  - `listing_id` 唯一索引
  - 小程序筛选索引

### 8.3 HPD Projection Service

```text
internal/domain/listingprojection/
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

约束：

- `domain/listingprojection` 只负责 HPD projection / lifecycle / `Apply(changes)`。
- `domain/listingprojection` 不对外提供 `house/search` 或 `house/public_detail` 读 API。
- handler 不直接调用 `domain/listingprojection` projector。

### 8.4 House Read Service

小程序找房读接口按“端侧 + API 模块”放在 `service/miniapp/house`：

```text
internal/service/miniapp/house/
  service.go
  search.go
  search_types.go
  detail.go
  detail_types.go
  mapper.go
  filter.go
  errors.go
```

文件职责：

- `service.go`
  - 定义 `HouseService` struct。
  - 定义 `NewHouseService`。
  - 定义 `miniappListingRepository` 接口，只暴露读接口：
    - `SearchMiniapp`
    - `CountMiniapp`
    - `FindOnlineDetail`
- `search.go`
  - 实现 `Search(ctx, input)`。
  - 负责分页归一化。
  - 调用 repository 查询列表和总数。
  - 返回 service 层业务结果，不返回 repository model。
- `search_types.go`
  - 定义 `SearchInput`。
  - 定义 `SearchResult`。
  - 定义 `ListItem`。
- `detail.go`
  - 实现 `GetPublicDetail(ctx, input)`。
  - 对非在线或不存在房源返回 `errcode.NotFound`。
- `detail_types.go`
  - 定义 `DetailInput`。
  - 定义 `DetailResult`。
  - 定义 `Detail`。
- `mapper.go`
  - `hpd.HpdMiniappListing -> ListItem`。
  - `hpd.HpdMiniappListing -> Detail`。
  - 负责 ObjectID 转 hex、枚举转 string、图片/费用对象复制。
- `filter.go`
  - `SearchInput -> repository/hpd.MiniappListingSearchFilter`。
  - 负责 skip / limit 计算。
  - 负责 asset mode、keyword、feature flags 等筛选字段透传到 repository filter。
- `errors.go`
  - 参数错误包装为 `errcode.InvalidParam`。
  - 详情不存在包装为 `errcode.NotFound`。
  - repository / Mongo 错误包装为 `errcode.DatabaseError`。

约束：

- `service/miniapp/house` 只能依赖 `repository/hpd.MiniappListingRepository`。
- `service/miniapp/house` 不能依赖 HMD repository / HMD service。
- `service/miniapp/house` 不能调用 `domain/listingprojection` projector。
- `service/miniapp/house` 返回业务结果，不直接返回 `hpd.HpdMiniappListing`。

### 8.5 House Handler

```text
internal/handler/v1/miniapp/house/
  handler.go
  routes.go
  search.go
  search_request.go
  search_response.go
  detail.go
  detail_request.go
  detail_response.go
```

文件职责：

- `handler.go`
  - 定义 `HouseHandler` struct。
  - 定义 `NewHouseHandler`。
  - 定义 handler 依赖的 `houseService` interface。
- `routes.go`
  - 注册 `POST /api/v1/house/search`。
  - 注册 `POST /api/v1/house/public_detail`。
- `search.go`
  - HTTP handler：绑定请求、转 service input、调用 `service/miniapp/house.Search`、返回响应。
- `search_request.go`
  - 定义 snake_case HTTP request。
  - 做 JSON bind 后的基础转换。
- `search_response.go`
  - 定义 snake_case API response DTO。
  - `service/miniapp/house.SearchResult -> API response`。
  - 分页响应必须是 `data.list/page/page_size/total`。
- `detail.go`
  - HTTP handler：绑定请求、解析 `listing_id` ObjectID、调用 `service/miniapp/house.GetPublicDetail`、返回响应。
- `detail_request.go`
  - 定义 `listing_id` 请求结构。
  - 做 ObjectID 解析。
- `detail_response.go`
  - 定义 snake_case API response DTO。
  - `service/miniapp/house.DetailResult -> API response`。

约束：

- handler 只做 JSON bind、ObjectID 解析、调 service、`response.Success` / `response.Err`。
- handler 不直接调用 `repository/hpd`。
- handler 不直接调用 `domain/listingprojection` projector。
- handler 不直接读取 HMD。
- 小程序 API 请求和响应字段统一使用 snake_case。
- 不得直接返回 Go model JSON。

## 9. Repository Filter 补充字段

当前 `repository/hpd.MiniappListingSearchFilter` 已有字段：

```text
city
district
biz_area
rent_mode
price range
skip
limit
```

需要补齐小程序 API 契约字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `asset_mode` | `hpd.HpdAssetMode` | `centralized` / `decentralized` |
| `keyword` | string | 关键词搜索 |
| `feature_flags` | `[]string` | 结构化筛选标记 |

筛选规则：

- `SearchMiniapp` 和 `CountMiniapp` 必须强制加 `is_online=1`。
- `asset_mode` 非空时必须校验合法枚举。
- `keyword` 匹配展示字段，第一期建议覆盖：
  - `title`
  - `community_name`
  - `building_or_community_name`
  - `address_text`
  - `subway_station`
- `feature_flags` 使用全部命中语义，建议 Mongo 条件为 `$all`。
- `FindOnlineDetail(listingID)` 必须同时匹配：
  - `listing_id`
  - `is_online=1`
  - `status=1`

## 10. Wire / 路由注册

新增 provider：

```text
wire/providers_house.go
```

建议内容：

```text
MiniappHouseSet = wire.NewSet(
  HpdRepositorySet 或 newHpdMiniappListingRepository,
  housesvc.NewHouseService,
  househandler.NewHouseHandler,
)
```

如果 `newHpdMiniappListingRepository` 已在 `providers_publish.go` 中定义，应考虑把 HPD repository provider 从 publish provider 中抽出，避免 house 依赖 publish set。

路由注册：

- `handler/v1/miniapp/house` 注册到 public `/api/v1` route group。
- `POST /api/v1/house/search` 是公开接口。
- `POST /api/v1/house/public_detail` 是公开接口。
- 不挂强制 auth middleware。

`public_detail` 后续需要支持“匿名可访问 + 带 token 返回收藏状态”时，应新增 optional auth 中间件或在 handler 内安全读取可选用户上下文；不能直接复用强制认证中间件。

## 11. 索引规划

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

如 `keyword` 需求确认用 Mongo 正则，第一期可先不加全文索引；后续如果搜索性能不足，再单独评估 text index 或搜索引擎。

## 12. 测试计划

### Repository

- `FindBySource`
- `UpsertBySource`
- `UpsertByListingID`
- `SearchMiniapp`
- `CountMiniapp`
- `FindOnlineDetail`
- `SearchMiniapp` 支持 `asset_mode`
- `SearchMiniapp` 支持 `keyword`
- `SearchMiniapp` 支持 `feature_flags`
- `SearchMiniapp` 永远只返回 `is_online=1`

### Projector

- 集中式房间生成 listing / miniapp_listing
- 分散式房间生成 listing / miniapp_listing
- 项目变更 fan-out 刷新集中式房间
- 楼栋变更 fan-out 刷新集中式房间
- 房型变更 fan-out 刷新引用房间
- 小区变更 fan-out 刷新分散式房间
- 分散式房间不依赖房型

### Service / House

- `Search` 参数校验：
  - page / page_size 归一化
  - 价格区间非法返回 `errcode.InvalidParam`
  - 非法枚举返回 `errcode.InvalidParam`
- `Search` 返回 `SearchResult`，不返回 repository model。
- `Search` 空结果返回空 `list` 和正确分页字段。
- `GetPublicDetail` 非法 `listing_id` 上游转换为参数错误。
- `GetPublicDetail` 未命中或非在线返回 `errcode.NotFound`。
- repository 错误包装为 `errcode.DatabaseError`。
- mapper 输出 service result 时不泄露内部字段：
  - `source_id`
  - `source_type`
  - `is_online`
  - `weight_score`

### Handler / House API

- `house/search` JSON bind。
- `house/search` 响应字段为 snake_case。
- `house/search` 分页响应结构为 `data.list/page/page_size/total`。
- `house/search` 不使用 `response.SuccessPage`。
- `house/public_detail` ObjectID 校验。
- `house/public_detail` 响应字段为 snake_case。
- `house/public_detail` 匿名访问返回 `is_favorited=false`。
- handler 不直接依赖 repository。

### E2E

- 发房端创建 HMD
- projector 生成 HPD
- 小程序查询列表命中
- 小程序详情返回展示字段

## 13. 实施顺序

已完成：

1. 定稿小程序查询 API 路径。
2. 补 `model/hpd*`。
3. 补 `repository/hpd` 基础能力。
4. 实现 `domain/listingprojection.Service.Apply(changes)` 的 miniapp projector。
5. 实现 HMD -> HPD miniapp read model 映射。

接下来：

1. 补 `repository/hpd.MiniappListingSearchFilter`：
   - `asset_mode`
   - `keyword`
   - `feature_flags`
2. 实现 `internal/service/miniapp/house`。
3. 实现 `internal/handler/v1/miniapp/house`。
4. Wire 接入 `MiniappHouseSet`。
5. 注册 `house/search`、`house/public_detail` 到 public `/api/v1` route group。
6. 补 repository / service / handler 测试。
7. 用 seed 或 fixture 创建 `listing_status=3` 的 HPD 数据并刷新 projection。
8. 用小程序接口查询 HPD 做 E2E。

## 14. 关键约束

- 小程序只读 HPD，不读 HMD。
- `domain/hmd.Service` 不直接依赖 HPD。
- projector 根据 HMD 当前源数据重建 HPD，不接收 handler 拼好的展示字段。
- `domain/listingprojection` 继续只负责 projection / lifecycle / `Apply(changes)`。
- `service/miniapp/house` 负责小程序找房读查询。
- `handler/v1/miniapp/house` 是小程序端 API 层；目录按端侧分组，路由仍按 API 模块 `house` 保持 `/api/v1/house/*`。
- 小程序接口请求和响应字段必须使用 snake_case。
- 不得直接返回 Go model JSON。
- `house/search` 分页响应必须放在 `data` 内，不使用 `response.SuccessPage`。
- 第一阶段不做集中式房型发布源。
- 第一阶段不做 outbox，先保证同步 projector 跑通。
- 第一阶段不做录入即发布，测试数据用 seed 灌入已上架状态。
- 不兼容旧数据库结构；字段以当前 schema 和当前前端确认需求为准。
