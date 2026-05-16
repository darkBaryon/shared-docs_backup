# 发房系统 API 接口文档

## 1. 适用范围

- 本文档是出房 Web 端第一期 API 契约源。
- 发房系统 API 不使用端侧名 `publish` 作为模块名；URL module 位统一使用业务对象名。
- 当前第一期只覆盖 HMD 录入与维护：集中式项目、楼栋、房型、集中式房间、分散式小区、分散式房间。
- 本文档定义的是对前端稳定暴露的 DTO，不直接等同 Go model、Mongo schema 或旧 Python 字段。

## 2. 通用约定

### 2.1 路由

路径命名遵守 [../overview/project-spec.md](../overview/project-spec.md)。

```text
POST /api/v1/{business_module}/{action}
```

约定：

- 所有业务接口统一使用 `POST`。
- `{business_module}` 使用业务对象名，例如 `centralized_project`、`building`、`room_type`、`centralized_room`、`decentralized_community`、`decentralized_room`。
- `{action}` 使用 snake_case，且必须是单个 path segment。
- 当 `{business_module}` 已明确业务对象时，`{action}` 可以使用 `create`、`update`、`detail`；涉及过滤或状态流转时使用更具体动作，例如 `list_by_project`、`update_status`。
- 不注册、不调用 `/api/v1/publish/{action}` 兼容路径。
- 页面路由可以多级组织，但后端 API 不使用 RESTful path。

### 2.2 字段命名

publish API request / response 字段统一使用 `snake_case`。

使用：

```text
project_name
room_type_id
room_status
listing_facilities
```

不要使用：

```text
projectName
roomTypeId
roomStatus
listingFacilities
```

约束：

- 后端不得直接把内部 Go model JSON 当作正式 publish API response。
- 前端 DTO 必须从本文档实现，不按 Go model、Mongo schema 或旧前端字段猜测生成。
- 前端页面 ViewModel 可以按组件需要转换，但转换必须集中维护，不能散落在页面中。

### 2.3 鉴权

除 `publish_auth/login` 外，publish 路由默认需要：

```text
Authorization: Bearer <token>
```

token 是后端签发的 opaque Redis session token。前端只原样放入 header，不解析 token 内容。

正式出房 Web 登录态使用 publish 专用 auth：

```text
POST /api/v1/publish_auth/login
POST /api/v1/publish_auth/session
POST /api/v1/publish_auth/logout
```

约束：

- `publish_auth` 只服务出房 Web，不复用小程序微信 auth。
- token 只是不透明 Redis session token，不是 JWT，不包含可供前端解析的权限信息。
- session 中的服务端 principal 是后端数据作用域判断依据；前端业务请求不得提交 `user_id`、`landlord_id`、`staff_id`、`owner_phone` 或 owner 过滤字段。
- HMD 不增加 owner/user/staff 归属字段；publish 房东归属通过 HPD root owner scope relation 表达。

### 2.4 响应包裹

成功：

```json
{
  "code": 0,
  "error": "",
  "data": {}
}
```

失败：

```json
{
  "code": 10001,
  "error": "参数无效",
  "data": null
}
```

约定：

- `code === 0` 表示业务成功。
- `error` 成功时为空字符串，失败时为可展示或可用于联调定位的错误信息。
- 前端不得依赖 HTTP status 判断业务成功，必须以 `code === 0` 为准。
- 资源不存在返回失败响应，`data = null`，不返回空对象。
- 空列表返回成功响应，`data.list = []`。

当前 publish 相关错误码：

| code | 含义 | 常见场景 |
| --- | --- | --- |
| `0` | 成功 | 业务成功 |
| `10001` | 参数无效 | JSON 绑定失败、必填字段缺失、ObjectID 非法、枚举非法 |
| `10002` | 未认证 | token 缺失、无效或过期 |
| `10003` | 无权限 | 非 local 环境使用本地手机号登录入口、无发房访问资格 |
| `10005` | 资源不存在 | 父级对象或目标对象不存在 |
| `10006` | 资源已存在 | 项目编码、楼栋编码、房号、房型名重复 |
| `50002` | 数据库错误 | Mongo 读写失败 |

### 2.5 Publish Auth

#### `PublishPrincipal`

`PublishPrincipal` 是 session principal 对外只读摘要。字段使用 `snake_case`。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `principal_type` | string | 是 | 固定为 `landlord` |
| `principal_id` | string | 是 | 当前身份 ObjectID hex |
| `terminal` | string | 是 | 固定为 `publish` |
| `phone` | string | 是 | 当前身份手机号 |
| `role_codes` | array<string> | 是 | 当前为空数组 |
| `permission_codes` | array<string> | 是 | 当前为空数组 |

#### `PublishAuthSubject`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 当前房东 landlord ID |
| `name` | string | 否 | 当前房东展示名 |
| `phone` | string | 是 | 当前房东手机号 |

#### `POST /api/v1/publish_auth/login`

鉴权：公开。

用途：出房 Web 登录并签发 Redis session token。

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `phone` | string | 是 | 房东登录手机号 |
| `password` | string | 是 | 账号密码；当前正式设计按密码登录 |

约束：

- 当前正式设计应使用 `hs_lld_landlord.phone + hs_lld_auth.password_hash` 做手机号密码登录。
- 当前代码里若仍存在 local 环境手机号直登，只视为联调临时实现，不作为正式契约。
- 不提交 `terminal`，该接口固定生成 `terminal=publish` 的 principal。
- 不提交 `user_id`、`landlord_id`、`staff_id`、owner 过滤字段。
- 登录后 publish 数据作用域按当前房东 `landlord` 名下 `project/community` owner scope 判断，而不是由前端传 owner 字段或由员工权限放大。

响应：

```ts
interface PublishAuthLoginResp {
  token: string
  principal: PublishPrincipal
  subject: PublishAuthSubject
}
```

返回：`ApiResponse<PublishAuthLoginResp>`

#### `POST /api/v1/publish_auth/session`

鉴权：需要 Bearer token。

用途：校验当前 token 并返回出房 Web 登录态。

请求：空对象 `{}`。

响应：

```ts
interface PublishAuthSessionResp {
  principal: PublishPrincipal
  subject: PublishAuthSubject
}
```

返回：`ApiResponse<PublishAuthSessionResp>`

约束：

- 只有 `terminal=publish` 的 session 可用于出房 Web。
- 前端只使用返回的 principal/subject 做展示和流程控制，不用它绕过后端权限判断。

#### `POST /api/v1/publish_auth/logout`

鉴权：需要 Bearer token。

用途：注销当前 Redis session token。

请求：空对象 `{}`。

响应：

```ts
interface PublishAuthLogoutResp {
  logged_out: boolean
}
```

返回：`ApiResponse<PublishAuthLogoutResp>`

### 2.6 数据作用域请求约束

所有 HMD 业务请求的数据作用域由后端从 session principal 推导，前端只提交业务筛选条件。

允许：

```text
city
district
project_name
community_name
```

禁止：

```text
user_id
landlord_id
staff_id
owner
owner_id
owner_phone
maintainer_staff_id
service_staff_id
```

### 2.7 列表与分页

第一期 `list_*` 接口不分页。

列表响应统一为：

```json
{
  "code": 0,
  "error": "",
  "data": {
    "list": []
  }
}
```

约束：

- 请求中不传 `page` / `page_size`。
- 后续确实需要分页时，先更新本文档，再调整前后端实现。
- 列表 item 当前使用对应业务对象的轻量 DTO；第一期轻量 DTO 与完整 entity 字段保持一致，方便列表页和选择器复用。
- 所有 `list_*` 默认排序统一为 `updated_at desc, id desc`，保证列表展示和前端测试稳定。

### 2.8 创建、更新和详情返回

- `create_*` 成功返回创建后的完整 entity。
- `update_*` 成功返回更新后的完整 entity。
- `*_detail` 成功返回完整 entity。
- `update_*_status` 成功返回更新后的完整 room entity。
- `update_*` 是表单全量保存，不是局部 patch；未提交的可选字段按空字符串、空数组、`0` 或 `null` 处理。

### 2.9 图片字段

楼栋：

```ts
photos: string[]
```

房型和房间：

```ts
images: Array<{
  url: string
  tag?: string
}>
```

约束：

- 楼栋照片只录入 URL 列表。
- 房型、集中式房间、分散式房间图片录入 URL + 图片标签。
- `tag` 使用 HMD schema 中 `image_tags` 枚举。
- 表单提交枚举值，不提交中文文案。

### 2.10 唯一性规则

| 对象 | 字段 | 唯一范围 | 说明 |
| --- | --- | --- | --- |
| 集中式项目 | `project_name + city` | 同一房东名下同城唯一 | 由 publish service 按当前房东 scope 校验，不是全局 Mongo 唯一索引 |
| 集中式楼栋 | `project_id + building_name` | 同一项目内唯一 | 不再使用 `building_code` |
| 集中式房型 | `room_type_name` | 同一项目内唯一；楼栋级房型还需在同一楼栋下唯一 | `project_id` / `building_id` 创建后不可更新 |
| 集中式房间 | `room_no` | 同一楼栋内唯一 | `project_id` / `building_id` / `room_type_id` 创建后不可更新 |
| 分散式小区 | `city + district + community_name` | 组合唯一 | `district` 为空时按空字符串参与组合唯一 |
| 分散式房间 | `room_no` | 同一分散式小区内唯一 | `decentralized_id` 创建后不可更新 |

## 3. 公共 DTO

### 3.1 `ApiResponse<T>`

```ts
interface ApiResponse<T> {
  code: number
  error: string
  data: T | null
}
```

### 3.2 `PublishListResp<T>`

```ts
interface PublishListResp<T> {
  list: T[]
}
```

### 3.3 `PublishEntityMeta`

所有 entity response 都带公共字段。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | Mongo ObjectID hex 字符串 |
| `created_at` | int64 | 是 | Unix 秒 |
| `updated_at` | int64 | 是 | Unix 秒 |
| `status` | int | 是 | 通用记录状态，`1` 有效，`-1` 删除/禁用 |
| `version` | int | 是 | 文档版本 |

### 3.4 `GeoPoint`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `lng` | number | 是 | 经度 |
| `lat` | number | 是 | 纬度 |

### 3.5 `TaggedImage`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `url` | string | 是 | 图片 URL |
| `tag` | string | 否 | 图片标签，取 `image_tags` 枚举 |

### 3.6 公共请求

#### `IDReq`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 目标对象 ID |

#### `ProjectIDReq`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `project_id` | string | 是 | 集中式项目 ID |

#### `BuildingIDReq`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `building_id` | string | 是 | 楼栋 ID |

#### `DecentralizedIDReq`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `decentralized_id` | string | 是 | 分散式小区主档 ID |

## 4. 枚举

枚举值以 [../schema/db-design/v4/modules/house-master-data.md](../schema/db-design/v4/modules/house-master-data.md) 为准。请求提交枚举值，不提交中文文案。

### 4.1 `rent_mode`

| 值 | 含义 |
| --- | --- |
| `whole` | 整租 |
| `shared` | 合租 |

### 4.2 `room_status`

`0` 只用于响应里的历史/异常未指定状态，不允许作为 `update_*_room_status` 的目标状态。

| 值 | 含义 |
| --- | --- |
| `0` | 未指定 |
| `1` | 未出租 |
| `2` | 已出租 |
| `3` | 已占用 |
| `-1` | 已下线/删除 |

### 4.3 其他字符串枚举

| 字段 | 可选值 |
| --- | --- |
| `decoration_level` | `rough`, `simple`, `fine`, `luxury` |
| `payment_cycle` | `monthly`, `quarterly`, `half_yearly`, `yearly`, `custom` |
| `agency_fee_mode` | `none`, `fixed`, `monthly_rent_multiple` |
| `viewing_time_rule` | `weekend_only`, `workday_only`, `anytime`, `workday_night_weekend` |
| `start_rent_rule` | `long_half_year`, `long_one_year`, `short_one_month`, `short_three_month`, `daily` |
| `orientation` | `east`, `south`, `west`, `north`, `southeast`, `northeast`, `southwest`, `northwest`, `south_north`, `unknown` |
| `listing_facilities` | `elevator`, `subway`, `convenience_store`, `parking`, `gym`, `activity_area`, `security_monitoring`, `book_bar`, `bar_counter`, `lounge`, `billiards`, `diy_dining_bar`, `laundry_room`, `table_soccer`, `sky_garden`, `cinema_area`, `front_desk`, `reception_area`, `dance_room`, `locker`, `pet_friendly` |
| `room_facilities` | `smart_lock`, `bed`, `wardrobe`, `desk_chair`, `heating`, `gas`, `broadband`, `tv`, `fridge`, `washing_machine`, `air_conditioner`, `water_heater`, `microwave`, `range_hood`, `induction_cooker`, `balcony`, `cooking_allowed`, `private_bathroom`, `sofa`, `water_purifier` |
| `image_tags` | `unknown`, `bedroom`, `master_bedroom`, `secondary_bedroom`, `living_room`, `dining_room`, `hallway`, `kitchen`, `bathroom`, `floor_plan_standard`, `floor_plan_non_standard`, `exterior` |

## 5. 集中式项目

### 5.1 `CentralizedProject`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 项目 ID |
| `project_name` | string | 是 | 项目名称 |
| `city` | string | 是 | 城市 |
| `district` | string | 否 | 区域 |
| `address_text` | string | 否 | 详细地址 |
| `geo` | `GeoPoint` | 否 | 坐标 |
| `brand_name` | string | 否 | 品牌名称 |
| `created_at` | int64 | 是 | Unix 秒 |
| `updated_at` | int64 | 是 | Unix 秒 |
| `status` | int | 是 | 通用记录状态 |
| `version` | int | 是 | 文档版本 |

`CentralizedProjectListItem = CentralizedProject`

### 5.2 `POST /api/v1/centralized_project/create`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `project_name` | string | 是 | 项目名称 |
| `city` | string | 是 | 城市 |
| `district` | string | 否 | 区域 |
| `address_text` | string | 否 | 详细地址 |
| `geo` | `GeoPoint` | 否 | 坐标 |
| `brand_name` | string | 否 | 品牌名称 |

响应：`ApiResponse<CentralizedProject>`

约束：

- 成功语义包含三步同时成功：
  1. 创建 `hs_hmd_centralized`
  2. 执行 `Apply(changes)` 同步 `hs_hpd_listing` / `hs_hpd_miniapp_listing` / `hs_hpd_publisher_listing`
  3. 创建当前房东对该项目的 `hs_hpd_root_scope_relation`（主归属键为 `owner_landlord_id`）
- 当前 Mongo 环境未启用事务，后端必须做应用层补偿一致性。
- 如果第 2 步或第 3 步失败，后端必须回滚第 1 步，不能留下“已创建但当前房东不可见”的孤儿项目。

### 5.3 `POST /api/v1/centralized_project/detail`

请求：`IDReq`

响应：`ApiResponse<CentralizedProject>`

### 5.4 `POST /api/v1/centralized_project/update`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 项目 ID |
| `project_name` | string | 是 | 项目名称 |
| `city` | string | 是 | 城市 |
| `district` | string | 否 | 区域 |
| `address_text` | string | 否 | 详细地址 |
| `geo` | `GeoPoint` | 否 | 坐标 |
| `brand_name` | string | 否 | 品牌名称 |

响应：`ApiResponse<CentralizedProject>`

### 5.5 `POST /api/v1/centralized_project/list`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `city` | string | 否 | 城市；不传表示不按城市筛选 |
| `district` | string | 否 | 区域；不传表示不按区域筛选 |

响应：`ApiResponse<PublishListResp<CentralizedProjectListItem>>`

## 6. 集中式楼栋

### 6.1 `Building`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 楼栋 ID |
| `project_id` | string | 是 | 项目 ID |
| `building_name` | string | 是 | 楼栋名称 |
| `floor_total` | int | 否 | 总层数 |
| `manager_name` | string | 否 | 管家姓名 |
| `manager_phone` | string | 否 | 管家电话 |
| `photos` | array<string> | 否 | 楼栋/公寓外观图 URL |
| `listing_facilities` | array<string> | 否 | 楼栋/小区配套枚举 |
| `created_at` | int64 | 是 | Unix 秒 |
| `updated_at` | int64 | 是 | Unix 秒 |
| `status` | int | 是 | 通用记录状态 |
| `version` | int | 是 | 文档版本 |

`BuildingListItem = Building`

### 6.2 `POST /api/v1/building/create`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `project_id` | string | 是 | 项目 ID |
| `building_name` | string | 是 | 楼栋名称 |
| `floor_total` | int | 否 | 总层数 |
| `manager_name` | string | 否 | 管家姓名 |
| `manager_phone` | string | 否 | 管家电话 |
| `photos` | array<string> | 否 | 楼栋/公寓外观图 URL |
| `listing_facilities` | array<string> | 否 | 楼栋/小区配套枚举 |

响应：`ApiResponse<Building>`

### 6.3 `POST /api/v1/building/detail`

请求：`IDReq`

响应：`ApiResponse<Building>`

### 6.4 `POST /api/v1/building/update`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 楼栋 ID |
| `building_name` | string | 是 | 楼栋名称 |
| `floor_total` | int | 否 | 总层数 |
| `manager_name` | string | 否 | 管家姓名 |
| `manager_phone` | string | 否 | 管家电话 |
| `photos` | array<string> | 否 | 楼栋/公寓外观图 URL |
| `listing_facilities` | array<string> | 否 | 楼栋/小区配套枚举 |

说明：`project_id` 创建后不可通过此接口更新。

响应：`ApiResponse<Building>`

### 6.5 `POST /api/v1/building/list_by_project`

请求：`ProjectIDReq`

响应：`ApiResponse<PublishListResp<BuildingListItem>>`

## 7. 集中式房型

### 7.1 `RoomType`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 房型 ID |
| `project_id` | string | 是 | 项目 ID |
| `building_id` | string | 否 | 楼栋 ID；项目级房型为空字符串 |
| `room_type_name` | string | 是 | 房型名称 |
| `room_count` | int | 否 | 室 |
| `hall_count` | int | 否 | 厅 |
| `bathroom_count` | int | 否 | 卫 |
| `kitchen_count` | int | 否 | 厨 |
| `area_size` | int | 否 | 面积，平方米 |
| `orientation` | string | 否 | 朝向枚举 |
| `decoration_level` | string | 否 | 装修情况枚举 |
| `payment_cycle` | string | 否 | 付租方式枚举 |
| `rent` | int | 否 | 月租金，单位元 |
| `deposit` | int | 否 | 押金，单位元 |
| `service_fee` | int | 否 | 服务费，单位元 |
| `agency_fee_mode` | string | 否 | 中介费模式枚举 |
| `agency_fee_value` | int | 否 | 中介费值 |
| `images` | array<`TaggedImage`> | 否 | 房型图片 |
| `room_facilities` | array<string> | 否 | 房间配置枚举 |
| `created_at` | int64 | 是 | Unix 秒 |
| `updated_at` | int64 | 是 | Unix 秒 |
| `status` | int | 是 | 通用记录状态 |
| `version` | int | 是 | 文档版本 |

`RoomTypeListItem = RoomType`

### 7.2 `POST /api/v1/room_type/create`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `project_id` | string | 否 | 项目 ID |
| `building_id` | string | 否 | 楼栋 ID |
| `room_type_name` | string | 是 | 房型名称 |
| `room_count` | int | 否 | 室 |
| `hall_count` | int | 否 | 厅 |
| `bathroom_count` | int | 否 | 卫 |
| `kitchen_count` | int | 否 | 厨 |
| `area_size` | int | 否 | 面积，平方米 |
| `orientation` | string | 否 | 朝向枚举 |
| `decoration_level` | string | 否 | 装修情况枚举 |
| `payment_cycle` | string | 否 | 付租方式枚举 |
| `rent` | int | 否 | 月租金，单位元 |
| `deposit` | int | 否 | 押金，单位元 |
| `service_fee` | int | 否 | 服务费，单位元 |
| `agency_fee_mode` | string | 否 | 中介费模式枚举 |
| `agency_fee_value` | int | 否 | 中介费值 |
| `images` | array<`TaggedImage`> | 否 | 房型图片 |
| `room_facilities` | array<string> | 否 | 房间配置枚举 |

约束：`project_id` 与 `building_id` 至少传一个；同时传入时，楼栋必须属于该项目。

响应：`ApiResponse<RoomType>`

### 7.3 `POST /api/v1/room_type/detail`

请求：`IDReq`

响应：`ApiResponse<RoomType>`

### 7.4 `POST /api/v1/room_type/update`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 房型 ID |
| `room_type_name` | string | 是 | 房型名称 |
| `room_count` | int | 否 | 室 |
| `hall_count` | int | 否 | 厅 |
| `bathroom_count` | int | 否 | 卫 |
| `kitchen_count` | int | 否 | 厨 |
| `area_size` | int | 否 | 面积，平方米 |
| `orientation` | string | 否 | 朝向枚举 |
| `decoration_level` | string | 否 | 装修情况枚举 |
| `payment_cycle` | string | 否 | 付租方式枚举 |
| `rent` | int | 否 | 月租金，单位元 |
| `deposit` | int | 否 | 押金，单位元 |
| `service_fee` | int | 否 | 服务费，单位元 |
| `agency_fee_mode` | string | 否 | 中介费模式枚举 |
| `agency_fee_value` | int | 否 | 中介费值 |
| `images` | array<`TaggedImage`> | 否 | 房型图片 |
| `room_facilities` | array<string> | 否 | 房间配置枚举 |

说明：`project_id`、`building_id` 创建后不可通过此接口更新。

响应：`ApiResponse<RoomType>`

### 7.5 `POST /api/v1/room_type/list_by_project`

请求：`ProjectIDReq`

响应：`ApiResponse<PublishListResp<RoomTypeListItem>>`

### 7.6 `POST /api/v1/room_type/list_by_building`

请求：`BuildingIDReq`

响应：`ApiResponse<PublishListResp<RoomTypeListItem>>`

## 8. 集中式房间

### 8.1 `CentralizedRoom`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 房间 ID |
| `project_id` | string | 是 | 项目 ID |
| `building_id` | string | 是 | 楼栋 ID |
| `room_type_id` | string | 否 | 房型 ID |
| `room_no` | string | 是 | 房间号 |
| `floor_no` | int | 否 | 楼层 |
| `rent_mode` | string | 是 | 租住方式枚举 |
| `layout_text` | string | 否 | 户型文案 |
| `area_size` | int | 否 | 面积，平方米 |
| `orientation` | string | 否 | 朝向枚举 |
| `decoration_level` | string | 否 | 装修情况枚举 |
| `payment_cycle` | string | 否 | 付租方式枚举 |
| `rent` | int | 否 | 月租金，单位元 |
| `deposit` | int | 否 | 押金，单位元 |
| `service_fee` | int | 否 | 服务费，单位元 |
| `agency_fee_mode` | string | 否 | 中介费模式枚举 |
| `agency_fee_value` | int | 否 | 中介费值 |
| `room_status` | int | 是 | 房态 |
| `viewing_time_rule` | string | 否 | 看房时间规则枚举 |
| `start_rent_rule` | string | 否 | 起租规则枚举 |
| `images` | array<`TaggedImage`> | 否 | 房间图片 |
| `room_facilities` | array<string> | 否 | 房间配置枚举 |
| `listing_facilities` | array<string> | 否 | 房源配置枚举 |
| `created_at` | int64 | 是 | Unix 秒 |
| `updated_at` | int64 | 是 | Unix 秒 |
| `status` | int | 是 | 通用记录状态 |
| `version` | int | 是 | 文档版本 |

`CentralizedRoomListItem = CentralizedRoom`

### 8.2 `POST /api/v1/centralized_room/create`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `project_id` | string | 是 | 项目 ID |
| `building_id` | string | 是 | 楼栋 ID |
| `room_type_id` | string | 否 | 房型 ID |
| `room_no` | string | 是 | 房间号 |
| `floor_no` | int | 否 | 楼层 |
| `rent_mode` | string | 是 | 租住方式枚举 |
| `layout_text` | string | 否 | 户型文案 |
| `area_size` | int | 否 | 面积，平方米 |
| `orientation` | string | 否 | 朝向枚举 |
| `decoration_level` | string | 否 | 装修情况枚举 |
| `payment_cycle` | string | 否 | 付租方式枚举 |
| `rent` | int | 否 | 月租金，单位元 |
| `deposit` | int | 否 | 押金，单位元 |
| `service_fee` | int | 否 | 服务费，单位元 |
| `agency_fee_mode` | string | 否 | 中介费模式枚举 |
| `agency_fee_value` | int | 否 | 中介费值 |
| `viewing_time_rule` | string | 否 | 看房时间规则枚举 |
| `start_rent_rule` | string | 否 | 起租规则枚举 |
| `images` | array<`TaggedImage`> | 否 | 房间图片 |
| `room_facilities` | array<string> | 否 | 房间配置枚举 |
| `listing_facilities` | array<string> | 否 | 房源配置枚举 |

约束：

- `building_id` 必须属于 `project_id`。
- `room_type_id` 可选；传入时必须属于同一项目或同一楼栋。
- 传入 `room_type_id` 时，若房间请求里未填写 `layout_text / area_size / orientation / decoration_level / payment_cycle / rent / deposit / service_fee / agency_fee_mode / agency_fee_value / images / room_facilities`，后端会自动用房型模板值补齐；其中 `layout_text` 会优先按房型的 `room_count / hall_count / bathroom_count / kitchen_count` 组装。
- 创建时 `room_status` 由后端默认写为 `1`，请求不提交。

响应：`ApiResponse<CentralizedRoom>`

### 8.3 `POST /api/v1/centralized_room/detail`

请求：`IDReq`

响应：`ApiResponse<CentralizedRoom>`

### 8.4 `POST /api/v1/centralized_room/update`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 房间 ID |
| `room_no` | string | 是 | 房间号 |
| `floor_no` | int | 否 | 楼层 |
| `rent_mode` | string | 是 | 租住方式枚举 |
| `layout_text` | string | 否 | 户型文案 |
| `area_size` | int | 否 | 面积，平方米 |
| `orientation` | string | 否 | 朝向枚举 |
| `decoration_level` | string | 否 | 装修情况枚举 |
| `payment_cycle` | string | 否 | 付租方式枚举 |
| `rent` | int | 否 | 月租金，单位元 |
| `deposit` | int | 否 | 押金，单位元 |
| `service_fee` | int | 否 | 服务费，单位元 |
| `agency_fee_mode` | string | 否 | 中介费模式枚举 |
| `agency_fee_value` | int | 否 | 中介费值 |
| `viewing_time_rule` | string | 否 | 看房时间规则枚举 |
| `start_rent_rule` | string | 否 | 起租规则枚举 |
| `images` | array<`TaggedImage`> | 否 | 房间图片 |
| `room_facilities` | array<string> | 否 | 房间配置枚举 |
| `listing_facilities` | array<string> | 否 | 房源配置枚举 |

说明：`project_id`、`building_id`、`room_type_id` 创建后不可通过此接口更新。

响应：`ApiResponse<CentralizedRoom>`

### 8.5 `POST /api/v1/centralized_room/update_status`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 房间 ID |
| `room_status` | int | 是 | 目标房态，只允许 `1` / `2` / `3` / `-1` |

响应：`ApiResponse<CentralizedRoom>`

### 8.6 `POST /api/v1/centralized_room/list_by_project`

请求：`ProjectIDReq`

响应：`ApiResponse<PublishListResp<CentralizedRoomListItem>>`

### 8.7 `POST /api/v1/centralized_room/list_by_building`

请求：`BuildingIDReq`

响应：`ApiResponse<PublishListResp<CentralizedRoomListItem>>`

## 9. 分散式小区

### 9.1 `DecentralizedCommunity`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 分散式小区主档 ID |
| `community_name` | string | 是 | 小区名称 |
| `city` | string | 是 | 城市 |
| `district` | string | 否 | 区域 |
| `biz_area` | string | 否 | 商圈 |
| `address_text` | string | 否 | 详细地址 |
| `geo` | `GeoPoint` | 否 | 坐标 |
| `subway_station` | string | 否 | 附近地铁站 |
| `created_at` | int64 | 是 | Unix 秒 |
| `updated_at` | int64 | 是 | Unix 秒 |
| `status` | int | 是 | 通用记录状态 |
| `version` | int | 是 | 文档版本 |

`DecentralizedCommunityListItem = DecentralizedCommunity`

### 9.2 `POST /api/v1/decentralized_community/create`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `community_name` | string | 是 | 小区名称 |
| `city` | string | 是 | 城市 |
| `district` | string | 否 | 区域 |
| `biz_area` | string | 否 | 商圈 |
| `address_text` | string | 否 | 详细地址 |
| `geo` | `GeoPoint` | 否 | 坐标 |
| `subway_station` | string | 否 | 附近地铁站 |

响应：`ApiResponse<DecentralizedCommunity>`

约束：

- 成功语义包含三步同时成功：
  1. 创建 `hs_hmd_decentralized`
  2. 执行 `Apply(changes)` 同步 `hs_hpd_listing` / `hs_hpd_miniapp_listing` / `hs_hpd_publisher_listing`
  3. 创建当前房东对该小区的 `hs_hpd_root_scope_relation`（主归属键为 `owner_landlord_id`）
- 当前 Mongo 环境未启用事务，后端必须做应用层补偿一致性。
- 如果第 2 步或第 3 步失败，后端必须回滚第 1 步，不能留下“已创建但当前房东不可见”的孤儿小区。

### 9.3 `POST /api/v1/decentralized_community/detail`

请求：`IDReq`

响应：`ApiResponse<DecentralizedCommunity>`

### 9.4 `POST /api/v1/decentralized_community/update`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 分散式小区主档 ID |
| `community_name` | string | 是 | 小区名称 |
| `city` | string | 是 | 城市 |
| `district` | string | 否 | 区域 |
| `biz_area` | string | 否 | 商圈 |
| `address_text` | string | 否 | 详细地址 |
| `geo` | `GeoPoint` | 否 | 坐标 |
| `subway_station` | string | 否 | 附近地铁站 |

响应：`ApiResponse<DecentralizedCommunity>`

### 9.5 `POST /api/v1/decentralized_community/list`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `city` | string | 否 | 城市；不传表示按当前房东全部可见小区返回 |
| `district` | string | 否 | 区域；仅作为当前房东可见小区范围内的附加筛选 |

响应：`ApiResponse<PublishListResp<DecentralizedCommunityListItem>>`

## 10. 分散式房间

### 10.1 `DecentralizedRoom`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 房间 ID |
| `decentralized_id` | string | 是 | 分散式小区主档 ID |
| `room_no` | string | 是 | 房间号 |
| `floor_no` | int | 否 | 楼层 |
| `rent_mode` | string | 是 | 租住方式枚举 |
| `layout_text` | string | 否 | 户型文案 |
| `area_size` | int | 否 | 面积，平方米 |
| `orientation` | string | 否 | 朝向枚举 |
| `decoration_level` | string | 否 | 装修情况枚举 |
| `payment_cycle` | string | 否 | 付租方式枚举 |
| `rent` | int | 否 | 月租金，单位元 |
| `deposit` | int | 否 | 押金，单位元 |
| `service_fee` | int | 否 | 服务费，单位元 |
| `agency_fee_mode` | string | 否 | 中介费模式枚举 |
| `agency_fee_value` | int | 否 | 中介费值 |
| `room_status` | int | 是 | 房态 |
| `viewing_time_rule` | string | 否 | 看房时间规则枚举 |
| `start_rent_rule` | string | 否 | 起租规则枚举 |
| `images` | array<`TaggedImage`> | 否 | 房间图片 |
| `room_facilities` | array<string> | 否 | 房间配置枚举 |
| `listing_facilities` | array<string> | 否 | 房源配置枚举 |
| `created_at` | int64 | 是 | Unix 秒 |
| `updated_at` | int64 | 是 | Unix 秒 |
| `status` | int | 是 | 通用记录状态 |
| `version` | int | 是 | 文档版本 |

`DecentralizedRoomListItem = DecentralizedRoom`

约束：分散式房间当前没有房型模型，request/response 均不得出现 `room_type_id`。

### 10.2 `POST /api/v1/decentralized_room/create`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `decentralized_id` | string | 是 | 分散式小区主档 ID |
| `room_no` | string | 是 | 房间号 |
| `floor_no` | int | 否 | 楼层 |
| `rent_mode` | string | 是 | 租住方式枚举 |
| `layout_text` | string | 否 | 户型文案 |
| `area_size` | int | 否 | 面积，平方米 |
| `orientation` | string | 否 | 朝向枚举 |
| `decoration_level` | string | 否 | 装修情况枚举 |
| `payment_cycle` | string | 否 | 付租方式枚举 |
| `rent` | int | 否 | 月租金，单位元 |
| `deposit` | int | 否 | 押金，单位元 |
| `service_fee` | int | 否 | 服务费，单位元 |
| `agency_fee_mode` | string | 否 | 中介费模式枚举 |
| `agency_fee_value` | int | 否 | 中介费值 |
| `viewing_time_rule` | string | 否 | 看房时间规则枚举 |
| `start_rent_rule` | string | 否 | 起租规则枚举 |
| `images` | array<`TaggedImage`> | 否 | 房间图片 |
| `room_facilities` | array<string> | 否 | 房间配置枚举 |
| `listing_facilities` | array<string> | 否 | 房源配置枚举 |

说明：创建时 `room_status` 由后端默认写为 `1`，请求不提交。

响应：`ApiResponse<DecentralizedRoom>`

### 10.3 `POST /api/v1/decentralized_room/detail`

请求：`IDReq`

响应：`ApiResponse<DecentralizedRoom>`

### 10.4 `POST /api/v1/decentralized_room/update`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 房间 ID |
| `room_no` | string | 是 | 房间号 |
| `floor_no` | int | 否 | 楼层 |
| `rent_mode` | string | 是 | 租住方式枚举 |
| `layout_text` | string | 否 | 户型文案 |
| `area_size` | int | 否 | 面积，平方米 |
| `orientation` | string | 否 | 朝向枚举 |
| `decoration_level` | string | 否 | 装修情况枚举 |
| `payment_cycle` | string | 否 | 付租方式枚举 |
| `rent` | int | 否 | 月租金，单位元 |
| `deposit` | int | 否 | 押金，单位元 |
| `service_fee` | int | 否 | 服务费，单位元 |
| `agency_fee_mode` | string | 否 | 中介费模式枚举 |
| `agency_fee_value` | int | 否 | 中介费值 |
| `viewing_time_rule` | string | 否 | 看房时间规则枚举 |
| `start_rent_rule` | string | 否 | 起租规则枚举 |
| `images` | array<`TaggedImage`> | 否 | 房间图片 |
| `room_facilities` | array<string> | 否 | 房间配置枚举 |
| `listing_facilities` | array<string> | 否 | 房源配置枚举 |

说明：`decentralized_id` 创建后不可通过此接口更新。

响应：`ApiResponse<DecentralizedRoom>`

### 10.5 `POST /api/v1/decentralized_room/update_status`

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | 是 | 房间 ID |
| `room_status` | int | 是 | 目标房态，只允许 `1` / `2` / `3` / `-1` |

响应：`ApiResponse<DecentralizedRoom>`

### 10.6 `POST /api/v1/decentralized_room/list_by_community`

请求：`DecentralizedIDReq`

响应：`ApiResponse<PublishListResp<DecentralizedRoomListItem>>`

## 11. 后续扩展

后续发房系统扩展到 HPD 时，仍继续沿用：

```text
POST /api/v1/{business_module}/{action}
```

例如未来可新增：

- `POST /api/v1/listing/create`
- `POST /api/v1/listing/update_publish_content`
- `POST /api/v1/listing/submit_for_audit`
- `POST /api/v1/listing/offline`

具体接口必须先更新本文档，再进入前端 DTO 和页面实现。
