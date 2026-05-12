# Customer MP HPD Frontend Migration Plan

本文档记录 `customer_mp` 小程序前端切换到 HPD 展示层的修改计划。

依据版本：

- `shared-docs`：`2714daf docs: clarify miniapp API implementation constraints`
- 后端规划：[../backend/miniapp-hpd.md](../backend/miniapp-hpd.md)
- HPD 字段：[../schema/db-design/v4/modules/house-publish-and-display.md](../schema/db-design/v4/modules/house-publish-and-display.md)
- 小程序接口：[../api/miniapp-api.md](../api/miniapp-api.md)

## 1. 当前阶段判断

当前小程序前端已经接入旧找房链路：

- `POST /api/v1/house/search`
- `POST /api/v1/house/public_detail`
- `POST /api/v1/favorite/add`
- `POST /api/v1/favorite/remove`
- `POST /api/v1/favorite/list`
- `POST /api/v1/history/add`
- `POST /api/v1/history/list`

但字段口径仍是旧模型：

- 房源主标识仍使用 `house_id`
- 列表和详情仍使用 `area`、`location`、`type`、`status`、`images`
- 搜索筛选仍包含旧字段 `area`、`room_count_min`、`near_subway`、`pet_friendly`、`has_elevator`、`cooking_allowed`、`furnished_level`
- 收藏、足迹、AI 房源卡片仍消费旧房源结构
- 请求错误提示仍优先读取 `detail/message/msg`，未优先读取 Go 后端的 `error`
- 分页仍使用 `limit`，未切到小程序 API 统一的 `page_size`

当前阶段应定义为：

> 小程序页面结构可复用，但服务层字段模型需要从旧 house 可见字段切换为 HPD 小程序 read model。

## 2. 要实现的需求

### 2.1 当前阶段要做

本阶段只做小程序读链路字段切换，不做后端 HPD projector。

目标：

- 小程序找房列表消费 `hs_hpd_miniapp_listing` 对外返回字段
- 小程序详情消费 `hs_hpd_miniapp_listing` 对外返回字段
- 前端内部统一使用 `listing_id` 作为房源展示 ID
- 收藏、取消收藏、足迹新增、收藏列表、足迹列表统一按 `listing_id` 语义处理
- AI 返回的房源卡片统一按 HPD 展示字段适配
- 请求错误处理优先读取 `{ code, error, data }` 中的 `error`
- 保留现有页面交互，不在本阶段重做 UI

小程序继续使用已确认接口路径：

```text
POST /api/v1/house/search
POST /api/v1/house/public_detail
```

字段来源以后端返回的 HPD 小程序 read model 为准：

```text
hs_hpd_miniapp_listing
```

核心字段以 [../api/miniapp-api.md](../api/miniapp-api.md) 的 `MiniappListingItem` 和 `MiniappListingDetail` 为前端接口契约。

主要字段：

- `listing_id`
- `asset_mode`
- `rent_mode`
- `city`
- `district`
- `biz_area`
- `community_name`
- `building_or_community_name`
- `subway_station`
- `subway_distance_m`
- `address_text`
- `geo`
- `title`
- `subtitle`
- `price`
- `price_text`
- `layout_text`
- `area_size`
- `orientation`
- `floor_text`
- `payment_cycle`
- `feature_flags`
- `listing_facilities`
- `platform_tags`
- `start_rent_rule`
- `cost_items`
- `description`
- `risk_notice`
- `contact_phone`
- `images`

不作为前端消费字段：

- `source_type`
- `source_id`
- `weight_score`
- `is_online`

说明：

- 上述字段属于 HPD 内部投影、排查或后端过滤/排序字段。
- 小程序 API 当前不返回这些字段。
- `is_online` 由后端在 `house/search` 和 `house/public_detail` 中保证：接口只返回在线房源，前端不再读取或展示 `is_online`。

### 2.2 当前阶段不做

本阶段不做：

- 不兼容旧 Python 接口返回结构
- 不保留旧 `house_id` 作为前端主标识
- 不让小程序直接依赖 HMD 字段
- 不新增 `/api/v1/miniapp/listing/search` 之类路径
- 不实现后台管理 HPD
- 不实现房东端 HPD
- 不实现发房端录入 UI
- 不重做页面视觉
- 不把旧 `status=1/2/0` 展示逻辑继续作为小程序主状态

说明：

- 若后端接口请求体字段还叫 `house_id`，需要同步改为 `listing_id`。
- 若后端为了临时联调返回旧字段，应视为后端未完成 HPD 对外契约，不在前端做兼容适配。

## 3. 实现思路

### 3.1 总体思路

前端页面不直接消费后端原始结构。

正确结构：

```text
pages
  -> services/*
    -> HPD raw payload
    -> page view model
```

服务层负责：

- 定义 HPD 原始类型
- 归一化图片、标签和价格文案
- 将 `listing_id` 映射为页面卡片 `id`
- 将 HPD 展示字段映射为页面已有 UI 所需字段

页面层负责：

- 搜索条件状态
- 卡片点击跳转
- 收藏、足迹、图片错误处理

### 3.2 ID 口径

统一口径：

```text
listing_id = hs_hpd_listing._id
```

强约束：

- 前端主链路只接受 `listing_id`。
- 不允许用 `_id`、`house_id` 或 `source_id` 作为 fallback。
- 若 `house/search`、`house/public_detail`、收藏、足迹或 AI 房源卡片返回项缺少 `listing_id`，前端 adapter 应直接视为后端契约错误并丢弃该项或抛出错误。
- `hs_hpd_miniapp_listing._id` 不是业务展示 ID，不能进入详情、收藏、足迹链路。

适用场景：

- 找房列表卡片 ID
- 详情页查询 ID
- 收藏 add/remove/list 关联 ID
- 历史 add/list 关联 ID
- AI 房源卡片跳转 ID

前端命名建议：

- 请求体字段：`listing_id`
- 页面变量可以继续叫 `houseId` 的地方应改成 `listingId`
- 类型名可继续保留 `HouseListItem` 这类页面语义名，但原始类型不应再叫旧 `RawHouseItem`

### 3.3 搜索参数口径

第一期建议参数：

- `city`
- `district`
- `biz_area`
- `min_price`
- `max_price`
- `rent_mode`
- `asset_mode`
- `keyword`
- `feature_flags`
- `page`
- `page_size`

后续可扩展：

- `listing_facilities`
- `payment_cycle`
- `sort_by`

旧参数处理：

- `area` 改为 `district`
- `near_subway` 改为 `feature_flags` 中的 `subway`
- `pet_friendly` 改为 `feature_flags` 中的 `pet_friendly`
- `has_elevator` 改为 `feature_flags` 中的 `elevator`
- `cooking_allowed` 改为 `feature_flags` 中的 `cooking_allowed`
- `room_count_min` 不继续作为第一期主筛选；如需要，后端应基于 `layout_text` 或新增结构化字段明确契约
- `furnished_level` 不继续使用；V4 HMD 使用 `decoration_level`，HPD 小程序 read model 当前未定义该字段

高级筛选第一期口径：

- 已进入 [../api/miniapp-api.md](../api/miniapp-api.md#86-feature_flags) 的高级筛选只提交 `feature_flags`。
- `feature_flags` 取值、展示文案和后端匹配语义以 [../api/miniapp-api.md](../api/miniapp-api.md#86-feature_flags) 为准。
- 前端可以保留近地铁、可养宠、电梯、可做饭等 UI，但必须映射为 `feature_flags` 数组。
- 没有进入 API 契约的筛选项必须隐藏、禁用或移除，不允许前端私自提交未确认字段。

## 4. 需要修改的前端文件

### 4.0 接口契约核对

文件：

- `shared-docs/api/miniapp-api.md`

修改：

- 开发前先确认 `house/search`、`house/public_detail`、`favorite/list`、`history/list` 的请求和响应结构已覆盖当前实现需要。
- 若实现发现字段缺口，先更新 `api/miniapp-api.md`，再改前端代码。
- AI 房源卡片响应结构当前未在 `api/miniapp-api.md` 中展开；实现 AI 卡片 HPD 化前，必须先补齐 `chat/send` 中房源卡片的响应契约。

原因：

- 前端 adapter 不能从 DB schema 猜接口 envelope。
- 小程序 API 已明确分页响应为 `data.list/page/page_size/total`，不得继续猜 `items/records/limit/has_more`。

### 4.1 请求工具

文件：

- `src/utils/request.ts`

修改：

- `ApiResponse` 增加 `error?: string`
- 非 2xx 和业务失败时优先读取 `error`
- 再降级读取 `detail/message/msg`

原因：

- Go 后端统一响应结构是 `{ code, error, data }`

### 4.2 找房服务层

文件：

- `src/services/house/types.ts`
- `src/services/house/adapters.ts`
- `src/services/house/payload.ts`
- `src/services/house/index.ts`

修改：

- 新增 HPD 原始类型，例如 `RawMiniappListing`
- 将 `RawHouseItem` 的旧字段替换为 HPD read model 字段
- ID 获取必须只读取 `listing_id`
- 缺少 `listing_id` 的列表项和详情响应都视为后端契约错误
- 联调期和开发态应直接抛错或显式提示契约错误，避免静默过滤导致分页 `total` 与展示列表不一致
- 生产态如确需降级，也应进入错误态或空态，并保留可观测日志，不作为普通 adapter 逻辑静默吞掉
- 图片归一化支持 `images: array<object>`，对象至少读取 `url`
- 价格展示优先使用 `price_text`
- 当 `price_text` 为空时，只能用 HPD 契约内必填的 `price` 和可选的 `payment_cycle` 生成展示文案
- 列表卡片使用：
  - `title`
  - `subtitle`
  - `district`
  - `biz_area`
  - `community_name`
  - `price_text`
  - `layout_text`
  - `platform_tags`
  - `listing_facilities`
- 详情使用：
  - `title`
  - `subtitle`
  - `price_text`
  - `layout_text`
  - `area_size`
  - `orientation`
  - `floor_text`
  - `payment_cycle`
  - `listing_facilities`
  - `description`
  - `risk_notice`
  - `contact_phone`
- 详情 adapter 同时消费 `data.house` 和 `data.is_favorited`
- `is_favorited` 写入详情页状态或 view model，用于首屏收藏按钮展示
- `public_detail` 请求体从 `house_id` 改为 `listing_id`
- 分页字段从 `limit` 改为 `page_size`
- 删除详情 fallback 里通过分页搜索反查旧房源的逻辑

### 4.3 找房页面

文件：

- `src/pages/discover/useDiscoverPage.ts`
- `src/pages/discover/components/DiscoverSearchPanel.vue`
- `src/pages/discover/discover.constants.ts`

修改：

- 页面筛选状态从 `selectedArea` 改为 `selectedDistrict`
- 请求参数从 `area` 改为 `district`
- 租住方式、价格、关键词、分页继续保留
- 旧布尔筛选统一映射为 `feature_flags`
- `furnished_level` 暂不展示或禁用
- 分页字段使用 `page_size`
- 不向后端提交未写入 `api/miniapp-api.md` 的筛选字段或未知 `feature_flags` 值

### 4.4 详情页

文件：

- `src/pages/house-detail/useHouseDetailPage.ts`
- `src/pages/house-detail/houseDetail.mapper.ts`
- `src/pages/house-detail/houseDetail.types.ts`

修改：

- 路由参数语义从 `houseId` 改为 `listingId`
- 详情 view model 使用 HPD 展示字段
- 信息项中移除 `房东 ID`
- 信息项改为展示：
  - `小区/项目`
  - `区域`
  - `商圈`
  - `面积`
  - `朝向`
  - `楼层`
  - `付款方式`
  - `起租要求`
  - `联系电话`
- 收藏和浏览历史调用传 `listing_id`
- 浏览历史新增必须同时传 `source`
- 详情页首次进入来源：
  - 从找房列表进入：`source=discover`
  - 从 AI 房源卡片进入：`source=ai`
  - 从详情页相关推荐或详情内部入口进入：`source=detail`
- 路由跳转建议携带 `source` 参数；若缺失，详情页默认按 `detail` 处理

### 4.5 收藏和历史

文件：

- `src/services/favorite/types.ts`
- `src/services/favorite/mapper.ts`
- `src/services/favorite/index.ts`
- `src/services/history/types.ts`
- `src/services/history/mapper.ts`
- `src/services/history/index.ts`
- `src/pages/favorites/useFavoritesPage.ts`
- `src/pages/history/useHistoryPage.ts`

修改：

- 请求体从 `house_id` 改为 `listing_id`
- 原始 item 支持 `listing_id`
- 嵌套房源结构改为 HPD listing 展示结构
- 卡片标题、价格、区域、封面统一复用 HPD adapter
- 详情补全逻辑改为按 `listing_id` 查详情
- 列表响应从旧 `favorites/history` 数组切换为统一分页结构 `data.list/page/page_size/total`
- `history/add` 请求体必须包含：
  - `listing_id`
  - `source`
- `source` 取值只允许 `discover`、`ai`、`detail`

### 4.6 AI 房源卡片

文件：

- `src/services/chat/types.ts`
- `src/services/chat/mapper.ts`

修改：

- AI 返回房源结构改为 HPD listing 字段
- 卡片 ID 使用 `listing_id`
- 标题使用 `title`
- 摘要优先使用 `subtitle` 或 `layout_text`
- 标签使用 `platform_tags` 和 `listing_facilities`
- 封面使用 HPD 图片归一化逻辑
- AI 房源卡片跳转详情时携带 `source=ai`

## 5. 实施顺序

推荐顺序：

1. 核对 [../api/miniapp-api.md](../api/miniapp-api.md)，确认当前要实现的请求体、响应 envelope 和字段已完整；如有缺口先补 API 文档
2. 更新 `request.ts` 错误响应读取逻辑
3. 新建或重写 `services/house` 的 HPD 类型和 adapter
4. 修改 `house/search` 和 `house/public_detail` 请求参数
5. 修改详情页 mapper 和路由变量命名
6. 修改收藏、历史的请求体和 mapper
7. 修改 discover 页面筛选参数，将高级筛选收敛到 `feature_flags`
8. 补齐 `chat/send` 的 HPD 房源卡片 API 契约，再修改 AI 房源卡片 mapper
9. 跑类型检查和构建
10. 使用后端 HPD fixture 做列表、详情、收藏、历史、AI 跳转联调

## 6. 验收标准

### 6.1 类型和构建

- TypeScript 类型检查通过
- 前端构建通过
- HPD raw type、house search payload、house detail mapper、favorite mapper、history mapper 中不再使用旧 `house_id`
- HPD raw type、house search payload、house detail mapper、favorite mapper、history mapper 中不再使用旧房源展示字段 `area/location/type/room_count/hall_count`
- `status` 只允许出现在请求状态、页面 loading 状态等非 HPD 房源主链路上下文中
- `area_size` 是 HPD 合法字段，不纳入旧字段禁用范围
- 找房、详情、收藏、足迹主链路中不存在 `_id` fallback
- 搜索分页请求使用 `page_size`

### 6.2 接口联调

- 找房列表请求命中 `POST /api/v1/house/search`
- 找房列表只展示在线房源，在线过滤由后端保证，前端不读取 `is_online`
- 卡片点击后使用 `listing_id` 进入详情
- 详情请求命中 `POST /api/v1/house/public_detail`
- 详情首屏收藏状态来自 `data.is_favorited`
- 收藏新增/取消使用 `listing_id`
- 浏览历史新增使用 `listing_id` 和正确 `source`
- 收藏列表和历史列表能展示 HPD 字段
- AI 返回房源卡片能跳转详情

### 6.3 行为边界

- 小程序前端不依赖 HMD 字段
- 小程序前端不兼容旧 Python 返回结构
- 小程序前端不自行推导发布状态
- 小程序前端不从旧房源字段反算 HPD 展示字段

## 7. 风险点

### 7.1 API 契约必须先于实现更新

当前 [../api/miniapp-api.md](../api/miniapp-api.md) 已经补齐找房、详情、收藏和足迹的基础请求响应结构。

仍需注意：

- AI 房源卡片响应结构尚未展开。
- 任何新增字段、分页结构变化、响应嵌套变化，都必须先更新 `api/miniapp-api.md`。

处理方式：

- 前端只按 `api/miniapp-api.md` 实现 adapter
- 不从 DB schema 直接猜接口返回 shape

### 7.2 收藏和历史接口必须使用 `listing_id`

现有代码仍使用 `house_id`。

处理方式：

- 当前计划明确切到 `listing_id`
- 若后端暂未改完，先改后端，不在前端做字段兼容

### 7.3 图片字段从字符串数组变为对象数组

HPD `images` 是 `array<object>`。

处理方式：

- 前端图片归一化只读取对象的 `url`
- 不再兼容纯字符串图片数组作为主契约

### 7.4 高级筛选按 `feature_flags` 落地

旧前端有近地铁、可养宠、电梯、可做饭、装修等级等筛选。

处理方式：

- 第一阶段已确认的配置类筛选统一走 `feature_flags`
- `feature_flags` 允许值以 [../api/miniapp-api.md](../api/miniapp-api.md#86-feature_flags) 为准
- 未写入 `api/miniapp-api.md` 的筛选项不提交
- `furnished_level` 暂不继续使用

## 8. 当前结论

这次前端改造不是页面重做，而是服务层契约切换。

优先改：

- ID：`house_id` -> `listing_id`
- 原始类型：旧 house -> HPD miniapp listing
- 搜索参数：旧区域/布尔字段 -> HPD 查询字段
- 错误响应：`message/detail` -> `error`

页面 UI 可以在第一阶段保持现状，等 HPD 接口 E2E 跑通后再按实际展示字段优化交互。
