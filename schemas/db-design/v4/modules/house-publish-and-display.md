# 房源发布与展示（V4）

## 1. 范围

- 本文档定义房源发布与前台展示层集合与字段。
- 公共字段见 [common-fields.md](./common-fields.md)。

## 2. 集合定义

### 2.1 `hs_hpd_listing`

用途：房源发布主实体（上架、下架、出租状态、展示内容）。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `source_system` | string | 是 | `"zfzf"` | 系统来源（运管列表展示） |
| `source_type` | string | 是 | 无 | 来源类型，关联主数据来源 |
| `source_id` | objectId | 是 | 无 | 来源对象 ID（按 source_type 指向） |
| `asset_mode` | string | 是 | 无 | 房源类型（centralized/decentralized） |
| `publish_title` | string | 是 | 无 | 展示标题 |
| `publish_subtitle` | string | 否 | `""` | 展示副标题 |
| `rent_mode` | string | 是 | 无 | 租住方式（whole/shared） |
| `listing_status` | int | 是 | `1` | 发布状态 |
| `audit_status` | int | 否 | `0` | 审核状态（运管展示） |
| `audit_remark` | string | 否 | `""` | 审核备注（运管展示） |
| `cover` | string | 否 | `""` | 封面图 URL |
| `gallery` | array<object> | 否 | `[]` | 展示图列表，建议 `[{url,tag}]` |
| `price` | int | 是 | `0` | 展示租金 |
| `price_min` | int | 否 | `0` | 价格区间下限（列表筛选） |
| `price_max` | int | 否 | `0` | 价格区间上限（列表筛选） |
| `layout_options` | array<string> | 否 | `[]` | 户型集合（列表筛选） |
| `rentable_count` | int | 否 | `0` | 可出租数量（审核列表展示） |
| `tags` | array<string> | 否 | `[]` | 业务标签 |
| `platform_tags` | array<string> | 否 | `[]` | 平台运营标签 |
| `weight_score` | int | 否 | `0` | 排序权重 |
| `contact_id` | objectId | 否 | 无 | 关联 `hs_hpd_contact._id` |
| `publisher_id` | objectId | 否 | 无 | 发房方 ID（关联发房方管理） |
| `publisher_name` | string | 否 | `""` | 发房方姓名/名称（运管展示） |
| `publisher_phone` | string | 否 | `""` | 发房方电话（运管展示） |
| `entrust_staff_id` | objectId | 否 | 无 | 租赁委托人（系统用户） |
| `entrust_staff_name` | string | 否 | `""` | 租赁委托人姓名（运管展示） |
| `city` | string | 否 | `""` | 城市（运管展示与筛选） |
| `district` | string | 否 | `""` | 区域（运管展示与筛选） |
| `biz_area` | string | 否 | `""` | 商圈（运管展示与筛选） |
| `subway_station` | string | 否 | `""` | 地铁站（运管展示） |
| `address_text` | string | 否 | `""` | 房源地址（运管展示） |
| `building_or_community_name` | string | 否 | `""` | 楼栋/小区名（运管展示） |
| `published_at` | int64 | 否 | `0` | 上架时间 |
| `offline_at` | int64 | 否 | `0` | 下架时间 |

索引：
- `source_type_1_source_id_1`
- `listing_status_1_updated_at_-1`
- `audit_status_1_updated_at_-1`
- `source_system_1_listing_status_1`
- `publisher_phone_1_status_1`
- `contact_id_1`

### 2.2 `hs_hpd_listing_index`

用途：前台检索与展示冗余索引（小程序列表与详情主读快照）。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `listing_id` | objectId | 是 | 无 | 关联 `hs_hpd_listing._id` |
| `city` | string | 是 | 无 | 城市 |
| `district` | string | 否 | `""` | 区域 |
| `biz_area` | string | 否 | `""` | 商圈 |
| `community_name` | string | 否 | `""` | 小区/项目名 |
| `rent_mode` | string | 是 | 无 | 整租/合租 |
| `subway_station` | string | 否 | `""` | 地铁站 |
| `subway_distance_m` | int | 否 | `0` | 距离地铁距离（米） |
| `address_text` | string | 否 | `""` | 地址文案（无地铁时展示） |
| `geo` | object | 否 | 无 | 坐标对象 `{lng,lat}` |
| `title` | string | 是 | 无 | 展示标题 |
| `subtitle` | string | 否 | `""` | 展示副标题 |
| `price` | int | 是 | `0` | 租金 |
| `price_text` | string | 否 | `""` | 价格文案 |
| `layout_text` | string | 否 | `""` | 户型文案 |
| `area_size` | int | 否 | `0` | 面积（详情展示） |
| `orientation` | string | 否 | `""` | 朝向（详情展示） |
| `floor_text` | string | 否 | `""` | 楼层文案（详情展示） |
| `payment_cycle` | string | 否 | `""` | 付租方式（详情展示） |
| `feature_flags` | array<string> | 否 | `[]` | 结构化筛选标记 |
| `listing_facilities` | array<string> | 否 | `[]` | 房源配置（详情展示） |
| `platform_tags` | array<string> | 否 | `[]` | 平台标签 |
| `start_rent_rule` | string | 否 | `""` | 起租要求（详情展示） |
| `cost_items` | array<object> | 否 | `[]` | 费用明细（详情展示） |
| `description` | string | 否 | `""` | 房源描述（详情展示） |
| `risk_notice` | string | 否 | `""` | 风险提示（详情展示） |
| `contact_phone` | string | 否 | `""` | 联系电话（详情展示） |
| `images` | array<object> | 否 | `[]` | 房源图片与房型/房间图片（详情展示） |
| `weight_score` | int | 否 | `0` | 排序权重 |
| `is_online` | int | 是 | `0` | 是否在线（0否1是） |

索引：
- `listing_id_1`（唯一）
- `is_online_1_city_1_district_1_price_1`
- `is_online_1_weight_score_-1_updated_at_-1`

### 2.3 `hs_hpd_contact`

用途：房源对外联系信息。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `contact_name` | string | 是 | 无 | 联系人名称 |
| `contact_phone` | string | 是 | 无 | 联系电话 |
| `contact_qr_code` | string | 否 | `""` | 含义待确认（见 TODO） |
| `contact_type` | string | 否 | `""` | 联系人类型 |
| `staff_id` | objectId | 否 | 无 | 关联员工（如客服） |

索引：
- `contact_phone_1`
- `staff_id_1_status_1`

### 2.4 `hs_hpd_entrust_relation`

用途：房源委托关系与服务归属。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `listing_id` | objectId | 是 | 无 | 关联 `hs_hpd_listing._id` |
| `owner_name` | string | 否 | `""` | 发房方/业主名称 |
| `owner_phone` | string | 否 | `""` | 发房方联系电话 |
| `maintainer_staff_id` | objectId | 否 | 无 | 维护员工 |
| `service_staff_id` | objectId | 否 | 无 | 服务员工 |
| `relation_status` | int | 是 | `1` | 委托关系状态 |
| `effective_from` | int64 | 否 | `0` | 生效时间 |
| `effective_to` | int64 | 否 | `0` | 失效时间 |

索引：
- `listing_id_1_status_1`
- `service_staff_id_1_status_1`
- `owner_phone_1_status_1`

## 3. 枚举字典

### 3.1 `source_type`

| 值 | 含义 |
| --- | --- |
| `decentralized_whole` | 来源为分散式整租房间 |
| `decentralized_shared` | 来源为分散式合租房间 |
| `centralized_room_type` | 来源为集中式房型模板 |
| `centralized_room` | 来源为集中式房间 |

适用字段：
- `hs_hpd_listing.source_type`

### 3.2 `listing_status`

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

### 3.3 `audit_status`

| 值 | 含义 |
| --- | --- |
| `0` | 未指定 |
| `1` | 待审核 |
| `2` | 审核通过 |
| `3` | 审核驳回 |
| `-1` | 已关闭 |

适用字段：
- `hs_hpd_listing.audit_status`

### 3.4 `contact_type`

| 值 | 含义 |
| --- | --- |
| `owner` | 业主/发房方 |
| `staff` | 平台员工 |
| `service` | 客服渠道 |

适用字段：
- `hs_hpd_contact.contact_type`

### 3.5 `relation_status`

| 值 | 含义 |
| --- | --- |
| `0` | 未指定 |
| `1` | 生效中 |
| `2` | 已结束 |
| `-1` | 已关闭 |

适用字段：
- `hs_hpd_entrust_relation.relation_status`

## 4. 读写约定

- 列表查询统一主读 `hs_hpd_listing_index`。
- 详情查询主读 `hs_hpd_listing_index`，必要时回源 `hs_hpd_listing` 与主数据补齐。
- `hs_hpd_listing` 更新后，需要同步刷新 `hs_hpd_listing_index`。

## 5. 字段来源映射（与主数据关联）

### 5.1 `hs_hpd_listing` 来源映射

| 字段 | 来源集合 | 来源字段 | 说明 |
| --- | --- | --- | --- |
| `source_type` | 系统写入 | - | 标记来源类型 |
| `source_id` | `hs_hmd_room_centralized` / `hs_hmd_room_decentralized` / `hs_hmd_room_type_centralized` | `_id` | 指向主数据源对象 |
| `rent_mode` | 主数据源对象 | `rent_mode` | 来源对象租住方式 |
| `price` | 主数据源对象 | `rent` | 展示租金默认取主数据租金 |
| `gallery` | 主数据源对象 | `images` | 展示图默认取主数据图片 |
| `publish_title` | 组装生成 | - | 基于主数据字段拼装 |
| `publish_subtitle` | 组装生成 | - | 基于主数据字段拼装 |
| `contact_id` | `hs_hpd_contact` | `_id` | 发布时绑定联系人 |
| `listing_status` | 发布流程 | - | 由发布/审核流程维护 |
| `platform_tags` | 运营配置 | - | 后台运营维护 |
| `weight_score` | 排序策略 | - | 规则计算或人工调整 |

### 5.2 `hs_hpd_listing_index` 来源映射

| 字段 | 来源集合 | 来源字段 | 说明 |
| --- | --- | --- | --- |
| `listing_id` | `hs_hpd_listing` | `_id` | 主记录关联键 |
| `title` | `hs_hpd_listing` | `publish_title` | 展示标题 |
| `subtitle` | `hs_hpd_listing` | `publish_subtitle` | 展示副标题 |
| `price` | `hs_hpd_listing` | `price` | 展示租金 |
| `platform_tags` | `hs_hpd_listing` | `platform_tags` | 平台标签 |
| `weight_score` | `hs_hpd_listing` | `weight_score` | 排序权重 |
| `is_online` | `hs_hpd_listing` | `listing_status` | `listing_status=3` 映射为在线 |
| `city` | 主数据集合 | `city` | 从 source_id 关联主数据读取 |
| `district` | 主数据集合 | `district` | 从 source_id 关联主数据读取 |
| `biz_area` | 主数据集合 | `biz_area` | 分散式来自 `hs_hmd_decentralized` |
| `community_name` | 主数据集合 | `community_name`/`project_name` | 按来源类型映射 |
| `subway_station` | 主数据集合 | `subway_station` | 主数据地铁站 |
| `geo` | 主数据集合 | `geo` | 坐标 |
| `layout_text` | 主数据集合 | `layout_text` / 房型字段组装 | 户型展示文案 |
| `feature_flags` | 主数据集合 | `listing_facilities`/`room_facilities` | 筛选标记来源于配置标签 |

## 6. 同步触发规则

- 新建 `listing`：必须同步生成 `listing_index`。
- 更新 `listing` 展示字段：同步更新 `listing_index`。
- 主数据变更（`source_id` 指向对象字段变化）：触发 `listing` 刷新并级联更新 `listing_index`。
- `listing_status` 变更：同步更新 `listing_index.is_online`。
