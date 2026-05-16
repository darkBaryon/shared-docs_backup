# 房源发布与展示（V4）

## 1. 范围

- 本文档定义房源发布与展示层集合与字段。
- HPD 不直接等同某一个前端页面；它包含发布主实体和各前端 read model。
- 当前第一期只落地小程序展示层。
- 当前第二期开始补发房端/房东端读模型。
- 公共字段见 [common-fields.md](./common-fields.md)。

## 2. 设计原则

HPD 分为两类数据：

```text
发布主实体：
  hs_hpd_listing

按前端/场景拆分的 read model：
  hs_hpd_miniapp_listing
  hs_hpd_admin_listing       // 后续设计，当前不建
  hs_hpd_publisher_listing
```

约定：

- `hs_hpd_listing` 只保存统一 listing identity、来源和发布状态。
- 小程序列表和详情只读 `hs_hpd_miniapp_listing`。
- 发房端/房东端读 `hs_hpd_publisher_listing`。
- 后台管理后续单独设计 read model，不复用小程序展示表。
- 收藏、足迹、预约、审核、运营推荐等跨模块引用统一使用 `hs_hpd_listing._id`。
- 不把后台审核字段、运管展示字段、小程序展示字段混在同一个 collection。

## 3. 集合定义

### 3.1 `hs_hpd_listing`

用途：房源发布主实体，提供稳定的 `listing_id`，承载来源、发布状态和生命周期。

它不是后台列表展示表，也不是小程序详情表。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `source_type` | string | 是 | 无 | 来源类型，稳定指向 HMD 来源集合 |
| `source_id` | objectId | 是 | 无 | 来源对象 ID |
| `asset_mode` | string | 是 | 无 | 房源类型：`centralized` / `decentralized` |
| `listing_status` | int | 是 | `1` | 发布状态，独立于 HMD `room_status` |
| `published_at` | int64 | 否 | `0` | 上架时间 |
| `offline_at` | int64 | 否 | `0` | 下架时间 |

索引：

- `source_type_1_source_id_1`（唯一）
- `listing_status_1_updated_at_-1`

### 3.2 `hs_hpd_miniapp_listing`

用途：小程序检索与展示 read model。小程序列表和详情第一期主读本集合。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `listing_id` | objectId | 是 | 无 | 关联 `hs_hpd_listing._id` |
| `source_type` | string | 是 | 无 | 冗余 `hs_hpd_listing.source_type`，便于排查和重建 |
| `source_id` | objectId | 是 | 无 | 冗余 `hs_hpd_listing.source_id` |
| `asset_mode` | string | 是 | 无 | 房源类型：`centralized` / `decentralized` |
| `rent_mode` | string | 是 | 无 | 租住方式：`whole` / `shared` |
| `city` | string | 是 | 无 | 城市 |
| `district` | string | 否 | `""` | 区域 |
| `biz_area` | string | 否 | `""` | 商圈 |
| `community_name` | string | 否 | `""` | 小区/项目名 |
| `building_or_community_name` | string | 否 | `""` | 楼栋/小区名 |
| `subway_station` | string | 否 | `""` | 地铁站 |
| `subway_distance_m` | int | 否 | `0` | 距离地铁距离，单位米 |
| `address_text` | string | 否 | `""` | 地址文案 |
| `geo` | object | 否 | 无 | 坐标对象 `{lng,lat}` |
| `title` | string | 是 | 无 | 展示标题 |
| `subtitle` | string | 否 | `""` | 展示副标题 |
| `price` | int | 是 | `0` | 租金 |
| `price_text` | string | 否 | `""` | 价格文案 |
| `layout_text` | string | 否 | `""` | 户型文案 |
| `area_size` | int | 否 | `0` | 面积 |
| `orientation` | string | 否 | `""` | 朝向 |
| `floor_text` | string | 否 | `""` | 楼层文案 |
| `payment_cycle` | string | 否 | `""` | 付租方式 |
| `feature_flags` | array<string> | 否 | `[]` | 结构化筛选标记 |
| `listing_facilities` | array<string> | 否 | `[]` | 房源配置 |
| `platform_tags` | array<string> | 否 | `[]` | 平台标签 |
| `start_rent_rule` | string | 否 | `""` | 起租要求 |
| `cost_items` | array<object> | 否 | `[]` | 费用明细 |
| `description` | string | 否 | `""` | 房源描述 |
| `risk_notice` | string | 否 | `""` | 风险提示 |
| `contact_phone` | string | 否 | `""` | 联系电话 |
| `images` | array<object> | 否 | `[]` | 房源图片 |
| `weight_score` | int | 否 | `0` | 排序权重 |
| `is_online` | int | 是 | `0` | 是否在线：0 否，1 是 |

索引：

- `listing_id_1`（唯一）
- `source_type_1_source_id_1`
- `is_online_1_city_1_district_1_price_1`
- `is_online_1_rent_mode_1_price_1`
- `is_online_1_weight_score_-1_updated_at_-1`

### 3.3 `hs_hpd_contact`

用途：房源对外联系信息。当前小程序 HPD 第一期不实现完整联系人链路，后续发布/审核流程接入时再落地。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `contact_name` | string | 是 | 无 | 联系人名称 |
| `contact_phone` | string | 是 | 无 | 联系电话 |
| `contact_qr_code` | string | 否 | `""` | 联系二维码 |
| `contact_type` | string | 否 | `""` | 联系人类型 |
| `staff_id` | objectId | 否 | 无 | 关联员工 |

索引：

- `contact_phone_1`
- `staff_id_1_status_1`

### 3.4 `hs_hpd_publisher_listing`

用途：发房端/房东端房源 read model。用于承载 publish 端房源列表、房源详情、状态浏览和后续筛选所需的聚合字段。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `listing_id` | objectId | 是 | 无 | 关联 `hs_hpd_listing._id` |
| `source_type` | string | 是 | 无 | 冗余 `hs_hpd_listing.source_type` |
| `source_id` | objectId | 是 | 无 | 冗余 `hs_hpd_listing.source_id` |
| `asset_mode` | string | 是 | 无 | 房源类型：`centralized` / `decentralized` |
| `owner_landlord_id` | objectId | 是 | 无 | 房东主体 `hs_lld_landlord._id`，便于发房端和后台按房东筛选 |
| `owner_phone_snapshot` | string | 否 | `""` | 房东手机号快照 |
| `landlord_name_snapshot` | string | 否 | `""` | 房东展示名快照；`hs_lld_profile` 未实现前可为空 |
| `root_type` | string | 是 | 无 | `centralized_project` / `decentralized_community` |
| `root_id` | objectId | 是 | 无 | 对应 root 主档 ID |
| `project_id` | objectId | 否 | 无 | 集中式项目 ID |
| `project_name` | string | 否 | `""` | 集中式项目名 |
| `building_id` | objectId | 否 | 无 | 楼栋 ID |
| `building_name` | string | 否 | `""` | 楼栋名 |
| `room_type_id` | objectId | 否 | 无 | 房型 ID |
| `room_type_name` | string | 否 | `""` | 房型名 |
| `decentralized_id` | objectId | 否 | 无 | 分散式小区 ID |
| `community_name` | string | 否 | `""` | 小区名 |
| `rent_mode` | string | 是 | 无 | 租住方式 |
| `city` | string | 是 | 无 | 城市 |
| `district` | string | 否 | `""` | 区域 |
| `biz_area` | string | 否 | `""` | 商圈 |
| `subway_station` | string | 否 | `""` | 地铁站 |
| `address_text` | string | 否 | `""` | 地址文案 |
| `geo` | object | 否 | 无 | 坐标对象 `{lng,lat}` |
| `room_no` | string | 是 | 无 | 房间号 |
| `floor_no` | int | 否 | `0` | 楼层 |
| `title` | string | 是 | 无 | publish 房源标题 |
| `subtitle` | string | 否 | `""` | publish 房源副标题 |
| `price` | int | 是 | `0` | 租金 |
| `price_text` | string | 否 | `""` | 价格文案 |
| `layout_text` | string | 否 | `""` | 户型文案 |
| `area_size` | int | 否 | `0` | 面积 |
| `orientation` | string | 否 | `""` | 朝向 |
| `decoration_level` | string | 否 | `""` | 装修情况 |
| `payment_cycle` | string | 否 | `""` | 付租方式 |
| `deposit` | int | 否 | `0` | 押金 |
| `service_fee` | int | 否 | `0` | 服务费 |
| `agency_fee_mode` | string | 否 | `""` | 中介费模式 |
| `agency_fee_value` | int | 否 | `0` | 中介费值 |
| `room_status` | int | 是 | 无 | HMD 房态 |
| `listing_status` | int | 是 | 无 | HPD 发布状态 |
| `viewing_time_rule` | string | 否 | `""` | 看房时间规则 |
| `start_rent_rule` | string | 否 | `""` | 起租规则 |
| `feature_flags` | array<string> | 否 | `[]` | 结构化展示/筛选标记 |
| `listing_facilities` | array<string> | 否 | `[]` | 房源配置 |
| `room_facilities` | array<string> | 否 | `[]` | 房间配置 |
| `images` | array<object> | 否 | `[]` | 房间图片 |
| `is_online` | int | 是 | `0` | 是否在线 |

索引：

- `listing_id_1`（唯一）
- `source_type_1_source_id_1`
- `owner_landlord_id_1_updated_at_-1`
- `root_type_1_root_id_1_updated_at_-1`
- `project_id_1_updated_at_-1`
- `building_id_1_updated_at_-1`
- `decentralized_id_1_updated_at_-1`
- `room_status_1_listing_status_1_updated_at_-1`

### 3.5 `hs_hpd_root_scope_relation`

用途：房东 root 主档归属关系。用于表达 `project/community -> owner_landlord_id`，支撑 publish 房东端的根作用域判断。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `root_type` | string | 是 | 无 | `centralized_project` / `decentralized_community` |
| `root_id` | objectId | 是 | 无 | 关联 HMD root 主档 `_id` |
| `owner_landlord_id` | objectId | 是 | 无 | 房东主体 `hs_lld_landlord._id` |
| `owner_phone` | string | 否 | `""` | 房东手机号快照，便于展示与排查 |
| `relation_status` | int | 是 | `1` | 归属关系状态 |
| `effective_from` | int64 | 否 | `0` | 生效时间 |
| `effective_to` | int64 | 否 | `0` | 失效时间 |

索引：

- `root_type_1_root_id_1_active_unique`（唯一，partial：`status=1 && relation_status=1`）
- `root_type_1_owner_landlord_id_1_relation_status_1_status_1`

## 4. 枚举字典

### 4.1 `source_type`

`source_type` 表示来源集合，不表示整租/合租。租住方式由 `rent_mode` 表示。

| 值 | 含义 |
| --- | --- |
| `centralized_room` | 来源为集中式房间实例 |
| `decentralized_room` | 来源为分散式房间实例 |

适用字段：

- `hs_hpd_listing.source_type`
- `hs_hpd_miniapp_listing.source_type`

### 4.2 `asset_mode`

| 值 | 含义 |
| --- | --- |
| `centralized` | 集中式 |
| `decentralized` | 分散式 |

适用字段：

- `hs_hpd_listing.asset_mode`
- `hs_hpd_miniapp_listing.asset_mode`

### 4.3 `listing_status`

| 值 | 含义 |
| --- | --- |
| `0` | 未指定 |
| `1` | 草稿 |
| `2` | 待审核 |
| `3` | 已上架 |
| `4` | 已下架 |
| `5` | 已出租 |
| `-1` | 已删除 |

适用字段：

- `hs_hpd_listing.listing_status`

### 4.4 `contact_type`

| 值 | 含义 |
| --- | --- |
| `owner` | 业主/发房方 |
| `staff` | 平台员工 |
| `service` | 客服渠道 |

适用字段：

- `hs_hpd_contact.contact_type`

### 4.5 `relation_status`

| 值 | 含义 |
| --- | --- |
| `0` | 未指定 |
| `1` | 生效中 |
| `2` | 已结束 |
| `-1` | 已关闭 |

适用字段：

- `hs_hpd_root_scope_relation.relation_status`

## 5. 读写约定

- 小程序列表查询主读 `hs_hpd_miniapp_listing`。
- 小程序详情查询主读 `hs_hpd_miniapp_listing`。
- 发房端房源列表与详情后续主读 `hs_hpd_publisher_listing`。
- 小程序接口不直接读取 HMD 集合。
- `hs_hpd_listing` 不承载前端展示字段，只作为统一 listing identity 和发布状态源。
- `hs_hpd_listing.listing_status` 是独立发布状态，不由 HMD `room_status` 自动推导。
- HMD `room_status` 是房间本身状态，只作为小程序展示安全门。
- 小程序可见性规则：`hs_hpd_listing.listing_status=3` 且 HMD `room_status=1` 时，`hs_hpd_miniapp_listing.is_online=1`；其他情况均为 `0`。
- `hs_hpd_listing.listing_status` 或 HMD `room_status` 变化后，必须刷新 `hs_hpd_miniapp_listing.is_online`。
- HMD 主数据变化后，必须按来源重建对应 `hs_hpd_miniapp_listing`。
- HMD 主数据变化后，也必须按来源重建对应 `hs_hpd_publisher_listing`。
- 后台管理与发房端/房东端 read model 后续单独设计，不从 `hs_hpd_miniapp_listing` 复用字段。
- 第一阶段不实现审核/发布动作时，projector 不自动把新房源置为已上架；测试和联调可通过 seed 创建 `listing_status=3` 的 HPD 数据。

### 5.1 小程序可见性映射

| `hs_hpd_listing.listing_status` | HMD `room_status` | `hs_hpd_miniapp_listing.is_online` | 说明 |
| --- | --- | --- | --- |
| `3` 已上架 | `1` 未出租/可租 | `1` | 小程序可展示 |
| `3` 已上架 | 非 `1` | `0` | 已发布但房间本身不可租，不展示 |
| 非 `3` | 任意 | `0` | 未发布、下架、已出租、删除等不展示 |

## 6. 字段来源映射

### 6.1 `hs_hpd_listing` 来源映射

| 字段 | 来源集合 | 来源字段 | 说明 |
| --- | --- | --- | --- |
| `source_type` | 系统写入 | - | 标记来源集合 |
| `source_id` | `hs_hmd_room_centralized` / `hs_hmd_room_decentralized` | `_id` | 指向 HMD 房间实例 |
| `asset_mode` | 系统写入 | - | 由来源集合决定 |
| `listing_status` | 发布流程 | - | 独立发布状态，不由 `room_status` 自动推导 |
| `published_at` | 发布流程 | - | 发布动作置为已上架时写入；第一阶段 seed 可直接写入 |
| `offline_at` | 发布流程 | - | 下架动作写入；第一阶段不由 projector 自动维护 |

### 6.2 `hs_hpd_miniapp_listing` 来源映射

| 字段 | 来源集合 | 来源字段 | 说明 |
| --- | --- | --- | --- |
| `listing_id` | `hs_hpd_listing` | `_id` | 统一 listing ID |
| `source_type` | `hs_hpd_listing` | `source_type` | 冗余来源类型 |
| `source_id` | `hs_hpd_listing` | `source_id` | 冗余来源 ID |
| `asset_mode` | `hs_hpd_listing` | `asset_mode` | 冗余房源类型 |
| `rent_mode` | HMD 房间集合 | `rent_mode` | 租住方式 |
| `is_online` | `hs_hpd_listing` + HMD 房态 | `listing_status` / `room_status` | `listing_status=3` 且 `room_status=1` 时为 `1` |
| `city` | HMD 项目/小区 | `city` | 集中式来自项目，分散式来自小区 |
| `district` | HMD 项目/小区 | `district` | 区域 |
| `biz_area` | `hs_hmd_decentralized` | `biz_area` | 分散式商圈 |
| `community_name` | HMD 项目/小区 | `project_name` / `community_name` | 小区/项目展示名 |
| `building_or_community_name` | HMD 楼栋/小区 | `building_name` / `community_name` | 楼栋/小区展示名 |
| `subway_station` | `hs_hmd_decentralized` | `subway_station` | 地铁站 |
| `address_text` | HMD 项目/小区 | `address_text` | 地址 |
| `geo` | HMD 项目/小区 | `geo` | 坐标 |
| `title` | 组装生成 | - | 基于项目/小区、楼栋、房间号 |
| `subtitle` | 组装生成 | - | 基于户型、面积、朝向、楼层 |
| `price` | HMD 房间集合 | `rent` | 展示租金 |
| `layout_text` | HMD 房间/房型 | `layout_text` / 房型字段 | 户型展示文案 |
| `area_size` | HMD 房间/房型 | `area_size` | 面积 |
| `orientation` | HMD 房间/房型 | `orientation` | 朝向 |
| `payment_cycle` | HMD 房间/房型 | `payment_cycle` | 付租方式 |
| `feature_flags` | HMD 配置标签 | `listing_facilities` / `room_facilities` | 筛选标记 |
| `listing_facilities` | HMD 配置标签 | `listing_facilities` / `room_facilities` | 详情配置展示 |
| `images` | HMD 房间/房型 | `images` | 房间图片优先，房型图片兜底 |

## 7. 同步触发规则

- 新建 HMD 房间：创建或复用 `hs_hpd_listing`，并生成 `hs_hpd_miniapp_listing`；不自动上架。
- 更新 HMD 房间展示字段：重建对应 `hs_hpd_miniapp_listing`。
- 更新 HMD 父级主数据：fan-out 重建受影响的 `hs_hpd_miniapp_listing`。
- 更新 `hs_hpd_listing.listing_status`：刷新对应 `hs_hpd_miniapp_listing.is_online`。
- 更新 HMD `room_status`：刷新对应 `hs_hpd_miniapp_listing.is_online`。
- 删除或软删除 HMD 来源对象：关闭小程序展示；是否同步变更 `hs_hpd_listing.listing_status` 由发布/管理流程决定。
