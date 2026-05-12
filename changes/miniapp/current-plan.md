# 小程序前端 API 对齐状态与修改计划

本文是小程序前端对接 Go 后端 miniapp API 的当前事实源。接口、字段和 ID 口径以 [../../api/miniapp-api.md](../../api/miniapp-api.md) 为准。

## 1. 当前项目怎么运作

小程序前端 API 调用分为四层：

```text
pages/*
  -> services/{module}
    -> api/{module}.ts
      -> utils/http / utils/request
        -> POST /api/v1/{module}/{action}
```

含义：

- `pages/*`：页面状态、交互和跳转。
- `services/{module}`：请求 payload、响应 DTO 和 view model mapper。
- `api/{module}.ts`：接口路径常量。
- `utils/request`：统一拼接 `API_BASE_URL + API_PREFIX`，注入 Bearer token，解析 `{ code, error, data }`。

当前重要约束：

- 小程序房源统一使用 `listing_id`，禁止继续使用旧 `house_id` 作为主链路字段。
- Miniapp API 请求和响应字段统一使用 `snake_case`。
- 前端只消费 miniapp API DTO，不直接消费 Go model JSON 或旧 Python 返回结构。
- HPD 房源展示只使用 `MiniappListingItem` / `MiniappListingDetail` 字段。

## 2. 当前已完成

### 2.1 请求层

已完成：

- 成功响应按 `code === 0` 判断。
- 失败响应读取 Go 后端 `error` 字段。
- token 以 `Authorization: Bearer <token>` 透传，不解析 token。

待注意：

- 当前 request 层仍会对所有错误弹 toast。对非阻断型上报接口，例如 history/add，页面层需要避免错误 toast 干扰主流程。

### 2.2 小程序 auth

已完成：

- `POST /api/v1/auth/wechat_login`
- `POST /api/v1/auth/wechat_register`

字段已对齐：

```text
wechat_login:    { code }
wechat_register: { code, phone_code }
```

当前状态：

- 微信登录失败返回未注册时，前端会展示微信手机号授权注册按钮。
- 注册成功后保存 opaque token，并继续拉取用户资料。

### 2.3 HPD 房源公开读链路

已完成：

- `POST /api/v1/house/search`
- `POST /api/v1/house/public_detail`

搜索请求字段已对齐：

```text
city
district
biz_area
rent_mode
asset_mode
min_price
max_price
keyword
feature_flags
page
page_size
```

详情请求字段已对齐：

```text
{ listing_id }
```

响应字段已按 HPD DTO 消费：

```text
listing_id
title
subtitle
price
price_text
layout_text
area_size
district
biz_area
community_name
building_or_community_name
platform_tags
listing_facilities
feature_flags
images[].url
is_favorited
```

### 2.4 feature_flags

第一期前端只提交 API 契约允许的值：

```text
subway
elevator
pet_friendly
cooking_allowed
```

旧筛选项只在能映射到上述字段时进入请求。不能映射到 API 契约的旧筛选，不应提交给后端。

### 2.5 收藏接口

已完成：

```text
POST /api/v1/favorite/add
POST /api/v1/favorite/remove
POST /api/v1/favorite/list
```

字段已对齐：

```text
favorite/add:    { listing_id }
favorite/remove: { listing_id }
favorite/list:   { page, page_size }
```

响应已对齐：

```text
favorite/list 读取 data.list
list item 使用 HPD MiniappListingItem
```

当前状态：

- 详情页收藏 / 取消收藏使用 `listing_id`。
- 收藏列表直接展示 HPD 卡片字段，不再回查 `public_detail` 兜旧结构。
- 不再使用 `_id` fallback。

### 2.6 足迹接口

已完成：

```text
POST /api/v1/history/add
POST /api/v1/history/list
```

字段已对齐：

```text
history/add:  { listing_id, source }
history/list: { page, page_size }
```

响应已对齐：

```text
history/list 读取 data.list
list item 使用 HPD MiniappListingItem + viewed_at
```

当前状态：

- 详情页加载成功后恢复足迹上报。
- history/add 失败不弹 toast，不打断详情页展示。
- source 映射：
  - 从找房列表进入详情：`discover`
  - 从 AI 推荐进入详情：`ai`
  - 从详情相关推荐进入详情：`detail`

### 2.7 用户接口类型

已完成：

```text
POST /api/v1/user/profile
POST /api/v1/user/update_profile
POST /api/v1/user/dashboard
```

类型已对齐：

```text
UserProfile:
user_id
nickname
avatar
phone
city
budget_min
budget_max
preferred_areas
preferred_rent_mode
move_in_plan
remark

UserDashboard:
favorite_count
history_count
plan_count
unread_notification_count
```

当前状态：

- 资料编辑页当前只提交 `nickname`、`avatar`。
- 前端 service 类型已删除旧 `role/contact/tenant/landlord/house_*` 字段。

### 2.8 旧 house 管理接口清理

已完成：

- 删除小程序前端 `HOUSE_API.detail/list/create/update/status/delete` 常量。
- 删除 `fetchMyHouseList/createHouse/updateHouseStatus` 旧 service。
- 房东页当前仍为本地表单缓存，不调用旧 house 管理接口。

## 3. 当前未完成

### 3.1 前端目标架构迁移

目标架构以 [../../frontend/development-spec.md](../../frontend/development-spec.md) 和 [../../frontend/miniapp.md](../../frontend/miniapp.md) 为准。

当前代码仍是旧结构：

```text
pages/*
  -> services/{module}
    -> api/{module}.ts  // URL 常量
      -> utils/http / utils/request
```

目标结构：

```text
pages/*
components/{module}
assets/css/{module}
api/{module}.ts      // URL + method + request type + response type + 调用函数
models/{model}.ts    // DTO -> ViewModel
utils/*
```

待迁移重点：

1. `api/*` 从 URL 常量升级为完整 API 函数。
2. `services/house/adapters.ts`、`services/favorite/mapper.ts`、`services/history/mapper.ts` 合并为统一 `models/listing.ts`。
3. `discover` 搜索面板 UI 状态回收到组件内，页面只接收 `search` 事件和查询参数。
4. 页面组件从 `pages/*` 迁到 `components/{module}`，样式迁到 `assets/css/{module}`。
5. `utils` 不再依赖 `pages` 常量，storage key 移到 `utils/storageKeys.ts` 或 `constants`。

迁移原则：

- 不一次性大重构。
- 每次只迁一个 feature 或一个 API 模块。
- 迁移后必须保持真机 E2E 可跑。

### 3.2 AI / chat 保持旧接口

当前决策：

- `chat/send` 本期明确继续使用旧接口，不迁移到 HPD miniapp API。
- chat 不纳入本轮 HPD API 字段对齐范围。

允许例外：

```text
src/services/chat
src/pages/ai
```

上述范围可以继续出现旧 `house_id/_id/area/location/room_count/hall_count`。

### 3.3 待真机验证

代码已完成字段对齐，但还需要用真机和 Go 后端测试数据跑完整链路：

```text
登录
找房列表
房源详情
收藏 / 取消收藏
收藏列表
足迹写入
足迹列表
个人资料
```

## 4. 修改顺序

### 4.1 已完成：收藏闭环

目标：

- 详情页收藏按钮可用。
- 收藏列表可展示 HPD 房源卡片。

已完成任务：

1. 修改 favorite types：raw list item 改为 HPD listing item。
2. 修改 `addFavorite` / `removeFavorite` 请求体为 `{ listing_id }`。
3. 修改 `fetchFavoriteList` 请求体为 `{ page, page_size }`。
4. 修改响应读取为 `data.list`。
5. favorite card mapper 复用 HPD 字段，禁止 `_id` fallback。
6. 待真机 E2E 验证：
   - 登录
   - 进入详情
   - 收藏
   - 进入收藏页
   - 取消收藏

### 4.2 已完成：足迹闭环

目标：

- 进入详情后记录浏览。
- 足迹列表可展示 HPD 房源卡片。

已完成任务：

1. 修改 history types：raw list item 改为 HPD listing item + `viewed_at`。
2. 修改 `addHistory` 请求体为 `{ listing_id, source }`。
3. 修改 `fetchHistoryList` 请求体为 `{ page, page_size }`。
4. 修改响应读取为 `data.list`。
5. 详情页按 query source 调用 `addHistory`。
6. AI 卡片跳详情时带 `source=ai`；列表跳详情带 `source=discover`。
7. history/add 失败静默或轻量上报，不弹阻断 toast。

### 4.3 已完成：用户资料

目标：

- 个人页和资料编辑页只使用当前 `UserProfile` / `UserDashboard` 字段。

已完成任务：

1. 修正 `UserProfile` 类型。
2. 修正 `UpdateUserProfilePayload` 类型。
3. 修正 `UserDashboard` 类型。
4. 检查 profile 页缓存结构，去掉旧 `contact/role` 依赖。

### 4.4 已决策：AI / chat 保持旧接口

目标：

- chat 不进入本轮 HPD miniapp API 字段迁移。

结论：

1. `POST /api/v1/chat/send` 继续使用旧接口。
2. `src/services/chat` 中旧字段不作为本轮检查失败项。
3. 详情跳转仍带 `source=ai`，用于 history/add。

### 4.5 已完成：旧 house 管理接口清理

目标：

- 避免旧接口被误认为当前 miniapp API。

已完成任务：

1. 删除或标注隔离 `HOUSE_API.detail/list/create/update/status/delete`。
2. 删除或隔离 `fetchMyHouseList/createHouse/updateHouseStatus`。
3. 房东页如需后端能力，重新走 publish/landlord 契约。

### 4.6 下一批：前端架构迁移

目标：

- 让 API、Model、Page、Component、Utils 五层边界清晰。

建议顺序：

1. `api/house.ts`：补 `searchHouses`、`getPublicHouseDetail` API 函数和 request/response 类型。
2. `models/listing.ts`：抽统一 HPD listing card mapper。
3. `api/favorite.ts`、`api/history.ts`：补 API 函数和 request/response 类型。
4. `favorite/history` 页面改为复用 `models/listing.ts`。
5. `discover` 搜索面板状态回收到组件内。
6. `components/discover`：迁移 discover 私有组件和常量，样式放入 `assets/css/discover`。
7. `utils/storageKeys.ts`：统一 token/profile/favorite/history cache key。

## 5. 验收标准

### 5.1 自动检查

以下范围内不应再出现旧房源主链路字段：

```text
src/services/favorite
src/services/history
src/pages/favorites
src/pages/history
```

禁止字段：

```text
house_id
_id fallback
area/location 作为房源展示字段
room_count/hall_count
images: string[] raw contract
limit 作为分页请求
favorites/history 作为列表响应 envelope
```

允许例外：

- UI 文案中的“区域”等中文展示词。
- 非房源主链路上下文中的普通状态字段。
- `src/services/chat` 和 `src/pages/ai`：本期明确保持旧接口。

### 5.2 真机 E2E

需要通过：

1. 微信登录。
2. 未注册时微信手机号注册。
3. 找房列表展示 HPD 房源。
4. 房源详情展示 HPD 图片、价格、标签、基础信息。
5. 收藏 / 取消收藏。
6. 收藏列表展示。
7. 进入详情写入足迹。
8. 足迹列表展示。

### 5.3 后端日志期望

E2E 过程中应出现：

```text
POST /api/v1/auth/wechat_login
POST /api/v1/auth/wechat_register
POST /api/v1/house/search
POST /api/v1/house/public_detail
POST /api/v1/favorite/add
POST /api/v1/favorite/remove
POST /api/v1/favorite/list
POST /api/v1/history/add
POST /api/v1/history/list
POST /api/v1/user/profile
POST /api/v1/user/update_profile
POST /api/v1/user/dashboard
```

不应出现：

```text
house_id
POST /api/v1/house/detail
POST /api/v1/house/list
POST /api/v1/house/create
POST /api/v1/house/status
```

## 6. 当前重要文档

日常只需要先读这些：

1. [../../README.md](../../README.md)：项目文档入口和阅读路径。
2. [../../api/miniapp-api.md](../../api/miniapp-api.md)：小程序 API 契约。
3. [../go_backend/current-plan.md](../go_backend/current-plan.md)：Go 后端当前进度。
4. [../../backend/miniapp-hpd.md](../../backend/miniapp-hpd.md)：小程序 HPD 展示层。
5. [../../schema/db-design/v4/index.md](../../schema/db-design/v4/index.md)：数据库设计入口。

如果这些文档和历史聊天记录冲突，以这些文档为准。

## 7. 文档维护规则

- 小程序前端 API 对齐状态只维护本文。
- 做完任务后更新本文的“已完成 / 未完成 / 修改顺序”。
- 已废弃接口、旧路径、旧数据库字段说明应从长期契约里删除，不在 changes 里扩散。
- 长期有效的 API 口径直接写入 [../../api/miniapp-api.md](../../api/miniapp-api.md)。
