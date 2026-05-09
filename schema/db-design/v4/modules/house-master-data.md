# 房源主数据（V4）

## 1. 范围

- 本文档定义房源主数据层集合与字段。
- 公共字段见 [common-fields.md](./common-fields.md)。

## 2. 集合定义

### 2.1 `hs_hmd_centralized`

用途：集中式主档。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `project_name` | string | 是 | 无 | 项目名称 |
| `project_code` | string | 是 | 无 | 项目编码（业务唯一） |
| `city` | string | 是 | 无 | 城市 |
| `district` | string | 否 | `""` | 区域 |
| `address_text` | string | 否 | `""` | 详细地址 |
| `geo` | object | 否 | 无 | 坐标对象 `{lng,lat}` |
| `brand_name` | string | 否 | `""` | 品牌名称 |

索引：
- `project_code_1`（唯一）
- `city_1_status_1`

### 2.2 `hs_hmd_building`

用途：楼栋主档。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `project_id` | objectId | 是 | 无 | 关联 `hs_hmd_centralized._id` |
| `building_name` | string | 是 | 无 | 楼栋名称 |
| `building_code` | string | 否 | `""` | 楼栋编码 |
| `floor_total` | int | 否 | `0` | 总层数 |
| `manager_name` | string | 否 | `""` | 楼栋管家姓名 |
| `manager_phone` | string | 否 | `""` | 楼栋管家电话 |
| `photos` | array<string> | 否 | `[]` | 楼栋/公寓外观图 |
| `listing_facilities` | array<string> | 否 | `[]` | 房源配置标签（楼栋/小区配套） |

索引：
- `project_id_1_status_1`
- `building_code_1`

### 2.3 `hs_hmd_decentralized`

用途：分散式主档。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `community_name` | string | 是 | 无 | 小区名称 |
| `city` | string | 是 | 无 | 城市 |
| `district` | string | 否 | `""` | 区域 |
| `biz_area` | string | 否 | `""` | 商圈 |
| `address_text` | string | 否 | `""` | 详细地址 |
| `geo` | object | 否 | 无 | 坐标对象 `{lng,lat}` |
| `subway_station` | string | 否 | `""` | 附近地铁站 |

索引：
- `city_1_district_1_community_name_1`
- `status_1_updated_at_-1`

### 2.4 `hs_hmd_room_type_centralized`

用途：集中式房型模板。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `project_id` | objectId | 否 | 无 | 关联项目 |
| `building_id` | objectId | 否 | 无 | 关联楼栋 |
| `room_type_name` | string | 是 | 无 | 房型名称 |
| `room_count` | int | 否 | `0` | 室 |
| `hall_count` | int | 否 | `0` | 厅 |
| `bathroom_count` | int | 否 | `0` | 卫 |
| `kitchen_count` | int | 否 | `0` | 厨 |
| `area_size` | int | 否 | `0` | 面积（平方米） |
| `orientation` | string | 否 | `""` | 朝向 |
| `decoration_level` | string | 否 | `""` | 装修情况 |
| `payment_cycle` | string | 否 | `""` | 付租方式 |
| `rent` | int | 否 | `0` | 租金 |
| `deposit` | int | 否 | `0` | 押金 |
| `service_fee` | int | 否 | `0` | 服务费 |
| `agency_fee_mode` | string | 否 | `""` | 中介费模式 |
| `agency_fee_value` | int | 否 | `0` | 中介费值 |
| `images` | array<string> | 否 | `[]` | 房型图片 |
| `room_facilities` | array<string> | 否 | `[]` | 房间配置标签 |

索引：
- `building_id_1_status_1`
- `project_id_1_room_type_name_1`
### 2.5 `hs_hmd_room_centralized`

用途：房间实例（可出租最小实体）。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `project_id` | objectId | 否 | 无 | 关联项目 |
| `building_id` | objectId | 否 | 无 | 关联楼栋 |
| `room_type_id` | objectId | 否 | 无 | 关联房型模板 |
| `room_no` | string | 是 | 无 | 房间号 |
| `floor_no` | int | 否 | `0` | 楼层 |
| `rent_mode` | string | 是 | 无 | 租住方式（whole/shared） |
| `layout_text` | string | 否 | `""` | 户型描述 |
| `area_size` | int | 否 | `0` | 面积 |
| `orientation` | string | 否 | `""` | 朝向 |
| `decoration_level` | string | 否 | `""` | 装修情况 |
| `payment_cycle` | string | 否 | `""` | 付租方式 |
| `rent` | int | 否 | `0` | 租金 |
| `deposit` | int | 否 | `0` | 押金 |
| `service_fee` | int | 否 | `0` | 服务费 |
| `agency_fee_mode` | string | 否 | `""` | 中介费模式 |
| `agency_fee_value` | int | 否 | `0` | 中介费值 |
| `room_status` | int | 否 | `1` | 房间状态 |
| `viewing_time_rule` | string | 否 | `""` | 看房时间规则 |
| `start_rent_rule` | string | 否 | `""` | 起租规则 |
| `images` | array<string> | 否 | `[]` | 房间图片 |
| `room_facilities` | array<string> | 否 | `[]` | 房间配置标签 |

索引：
- `building_id_1_room_no_1`（唯一）
- `status_1_updated_at_-1`

### 2.6 `hs_hmd_room_decentralized`

用途：分散式房间实例（可出租最小实体）。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `decentralized_id` | objectId | 是 | 无 | 关联 `hs_hmd_decentralized._id` |
| `room_no` | string | 是 | 无 | 房间号 |
| `floor_no` | int | 否 | `0` | 楼层 |
| `rent_mode` | string | 是 | 无 | 租住方式（whole/shared） |
| `layout_text` | string | 否 | `""` | 户型描述 |
| `area_size` | int | 否 | `0` | 面积 |
| `orientation` | string | 否 | `""` | 朝向 |
| `decoration_level` | string | 否 | `""` | 装修情况 |
| `payment_cycle` | string | 否 | `""` | 付租方式 |
| `rent` | int | 否 | `0` | 租金 |
| `deposit` | int | 否 | `0` | 押金 |
| `service_fee` | int | 否 | `0` | 服务费 |
| `agency_fee_mode` | string | 否 | `""` | 中介费模式 |
| `agency_fee_value` | int | 否 | `0` | 中介费值 |
| `room_status` | int | 否 | `1` | 房间状态 |
| `viewing_time_rule` | string | 否 | `""` | 看房时间规则 |
| `start_rent_rule` | string | 否 | `""` | 起租规则 |
| `images` | array<string> | 否 | `[]` | 房间图片 |
| `room_facilities` | array<string> | 否 | `[]` | 房间配置标签 |

当前约束：

- 分散式暂不提供独立房型模型
- `hs_hmd_room_decentralized` 当前不包含 `room_type_id`

索引：
- `decentralized_id_1_room_no_1`（唯一）
- `status_1_updated_at_-1`

## 3. 枚举字典

### 3.1 `rent_mode`

| 值 | 含义 |
| --- | --- |
| `whole` | 整租 |
| `shared` | 合租 |

适用字段：
- `hs_hmd_room_centralized.rent_mode`
- `hs_hmd_room_decentralized.rent_mode`

### 3.2 `room_status`

| 值 | 含义 |
| --- | --- |
| `0` | 未指定 |
| `1` | 未出租 |
| `2` | 已出租 |
| `3` | 已占用 |
| `-1` | 已下线/删除 |

适用字段：
- `hs_hmd_room_centralized.room_status`
- `hs_hmd_room_decentralized.room_status`

### 3.3 `decoration_level`

| 值 | 含义 |
| --- | --- |
| `rough` | 毛坯 |
| `simple` | 简单装修 |
| `fine` | 精装修 |
| `luxury` | 豪华装修 |

适用字段：
- `hs_hmd_room_type_centralized.decoration_level`
- `hs_hmd_room_centralized.decoration_level`
- `hs_hmd_room_decentralized.decoration_level`

### 3.4 `payment_cycle`

| 值 | 含义 |
| --- | --- |
| `monthly` | 月付 |
| `quarterly` | 季付 |
| `half_yearly` | 半年付 |
| `yearly` | 年付 |
| `custom` | 自定义 |

适用字段：
- `hs_hmd_room_type_centralized.payment_cycle`
- `hs_hmd_room_centralized.payment_cycle`
- `hs_hmd_room_decentralized.payment_cycle`

### 3.5 `agency_fee_mode`

| 值 | 含义 |
| --- | --- |
| `none` | 无中介费 |
| `fixed` | 固定金额 |
| `monthly_rent_multiple` | 按月租倍数 |

适用字段：
- `hs_hmd_room_type_centralized.agency_fee_mode`
- `hs_hmd_room_centralized.agency_fee_mode`
- `hs_hmd_room_decentralized.agency_fee_mode`

### 3.6 `viewing_time_rule`

| 值 | 含义 |
| --- | --- |
| `weekend_only` | 仅周末 |
| `workday_only` | 仅工作日 |
| `anytime` | 随时看房 |
| `workday_night_weekend` | 工作日晚上和周末 |

适用字段：
- `hs_hmd_room_centralized.viewing_time_rule`
- `hs_hmd_room_decentralized.viewing_time_rule`

### 3.7 `start_rent_rule`

| 值 | 含义 |
| --- | --- |
| `long_half_year` | 长租（半年起） |
| `long_one_year` | 长租（一年起） |
| `short_one_month` | 短租（一个月起） |
| `short_three_month` | 短租（三个月起） |
| `daily` | 日租 |

适用字段：
- `hs_hmd_room_centralized.start_rent_rule`
- `hs_hmd_room_decentralized.start_rent_rule`

### 3.8 `orientation`

| 值 | 含义 |
| --- | --- |
| `east` | 东 |
| `south` | 南 |
| `west` | 西 |
| `north` | 北 |
| `southeast` | 东南 |
| `northeast` | 东北 |
| `southwest` | 西南 |
| `northwest` | 西北 |
| `south_north` | 南北通透 |
| `unknown` | 未知 |

适用字段：
- `hs_hmd_room_type_centralized.orientation`
- `hs_hmd_room_centralized.orientation`
- `hs_hmd_room_decentralized.orientation`

### 3.9 `listing_facilities`（房源配置）

| 值 | 含义 |
| --- | --- |
| `elevator` | 电梯 |
| `subway` | 地铁 |
| `convenience_store` | 便利店/超市 |
| `parking` | 停车场 |
| `gym` | 健身房 |
| `activity_area` | 活动场地 |
| `security_monitoring` | 安全监控 |
| `book_bar` | 书吧 |
| `bar_counter` | 吧台 |
| `lounge` | 休闲区 |
| `billiards` | 桌球区 |
| `diy_dining_bar` | DIY 餐吧 |
| `laundry_room` | 洗衣房 |
| `table_soccer` | 桌上足球 |
| `sky_garden` | 空中花园 |
| `cinema_area` | 影视区 |
| `front_desk` | 前台 |
| `reception_area` | 会客区 |
| `dance_room` | 舞蹈室 |
| `locker` | 快递柜 |
| `pet_friendly` | 可养宠物 |

适用字段：
- `hs_hmd_building.listing_facilities`
- `hs_hmd_room_centralized.listing_facilities`
- `hs_hmd_room_decentralized.listing_facilities`

### 3.10 `room_facilities`（房间配置）

| 值 | 含义 |
| --- | --- |
| `smart_lock` | 智能锁 |
| `bed` | 床 |
| `wardrobe` | 衣柜 |
| `desk_chair` | 书桌椅 |
| `heating` | 暖气 |
| `gas` | 天然气 |
| `broadband` | 宽带 |
| `tv` | 电视 |
| `fridge` | 冰箱 |
| `washing_machine` | 洗衣机 |
| `air_conditioner` | 空调 |
| `water_heater` | 热水器 |
| `microwave` | 微波炉 |
| `range_hood` | 油烟机 |
| `induction_cooker` | 电磁炉 |
| `balcony` | 阳台 |
| `cooking_allowed` | 可做饭 |
| `private_bathroom` | 独立卫生间 |
| `sofa` | 沙发 |
| `water_purifier` | 净水器 |

适用字段：
- `hs_hmd_room_type_centralized.room_facilities`
- `hs_hmd_room_centralized.room_facilities`
- `hs_hmd_room_decentralized.room_facilities`

### 3.11 `image_tags`（房间照片分类）

| 值 | 含义 |
| --- | --- |
| `unknown` | 未知 |
| `bedroom` | 卧室 |
| `master_bedroom` | 主卧 |
| `secondary_bedroom` | 次卧 |
| `living_room` | 客厅 |
| `dining_room` | 餐厅 |
| `hallway` | 过厅 |
| `kitchen` | 厨房 |
| `bathroom` | 卫生间 |
| `floor_plan_standard` | 标准户型图 |
| `floor_plan_non_standard` | 非标准户型图 |
| `exterior` | 外景 |

适用字段：
- `hs_hmd_room_type_centralized.images`
- `hs_hmd_room_centralized.images`
- `hs_hmd_room_decentralized.images`

备注：
- `images` 字段建议由对象数组承载：`[{url, tag}]`，其中 `tag` 取上述枚举值。
