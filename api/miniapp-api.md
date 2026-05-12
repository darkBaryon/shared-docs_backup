# Miniapp API

本文档是小程序前端接口契约源。字段以当前 HPD / 用户模型为准，不兼容旧 Python 接口、旧数据库字段或旧前端字段。

## 1. 通用约定

### 1.1 路由

所有业务接口统一使用：

```text
POST /api/v1/{module}/{action}
```

小程序是前端端类型，不作为 API 模块名。找房归属 `house` 模块，用户行为归属 `favorite`、`history` 模块。

### 1.2 响应结构

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
  "error": "参数错误",
  "data": null
}
```

约定：

- `code === 0` 表示业务成功。
- `error` 成功时为空字符串，失败时为可展示错误文案。
- 前端不得依赖 HTTP status 判断业务成功。

### 1.3 字段命名

Miniapp API 请求和响应字段统一使用 `snake_case`。

后端不得直接返回 Go model JSON。当前 Go model 的 JSON tag 多为 `camelCase`，小程序接口必须通过 miniapp response DTO 映射输出，例如 `listingId` 映射为 `listing_id`、`assetMode` 映射为 `asset_mode`。

### 1.4 鉴权

公开接口：

- `POST /api/v1/auth/wechat_login`
- `POST /api/v1/auth/wechat_register`
- `POST /api/v1/house/search`
- `POST /api/v1/house/public_detail`

其他接口默认需要：

```text
Authorization: Bearer <token>
```

`house/public_detail` 允许匿名访问。匿名或无效 token 访问时 `is_favorited=false`；如果请求带有效 token，后端返回真实收藏状态。

token 是后端签发的 opaque Redis session token，不是 JWT。前端只需要原样放入 `Authorization: Bearer <token>`，不要解析 token 内容，也不要依赖 token 自身判断过期。

### 1.5 ID 口径

- 小程序房源统一使用 `listing_id`。
- `listing_id` 关联 `hs_hpd_listing._id`。
- 收藏、足迹、房源详情都使用同一个 `listing_id`。
- 禁止使用旧字段 `house_id`。

### 1.6 分页

分页请求统一使用：

| 字段 | 类型 | 必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| `page` | int | 否 | `1` | 从 1 开始 |
| `page_size` | int | 否 | `20` | 最大 50 |

分页响应统一放在 `data` 内：

```json
{
  "code": 0,
  "error": "",
  "data": {
    "list": [],
    "page": 1,
    "page_size": 20,
    "total": 0
  }
}
```

实现要求：小程序分页接口不得使用当前 Go 后端 `response.SuccessPage` 的顶层 `maxSize/size/data` 结构。实现时应使用符合本契约的 DTO 或 `response.Success(c, { list, page, page_size, total })`。

## 2. 公共对象

### 2.1 `MiniappListingItem`

用于房源列表、收藏列表、足迹列表的房源摘要。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `listing_id` | string | 是 | HPD 统一房源 ID |
| `asset_mode` | string | 是 | `centralized` 集中式；`decentralized` 分散式 |
| `rent_mode` | string | 是 | `whole` 整租；`shared` 合租 |
| `city` | string | 是 | 城市 |
| `district` | string | 否 | 区域 |
| `biz_area` | string | 否 | 商圈 |
| `community_name` | string | 否 | 项目/小区名 |
| `building_or_community_name` | string | 否 | 楼栋/小区展示名 |
| `subway_station` | string | 否 | 地铁站 |
| `subway_distance_m` | int | 否 | 距离地铁站米数 |
| `title` | string | 是 | 列表标题 |
| `subtitle` | string | 否 | 副标题 |
| `price` | int | 是 | 月租金，单位元 |
| `price_text` | string | 否 | 展示价格文案 |
| `layout_text` | string | 否 | 户型文案 |
| `area_size` | int | 否 | 面积，单位平方米 |
| `orientation` | string | 否 | 朝向枚举 |
| `floor_text` | string | 否 | 楼层文案 |
| `payment_cycle` | string | 否 | `monthly` / `quarterly` / `half_yearly` / `yearly` / `custom` |
| `feature_flags` | array<string> | 否 | 结构化筛选标记 |
| `listing_facilities` | array<string> | 否 | 房源配置 |
| `platform_tags` | array<string> | 否 | 平台展示标签 |
| `images` | array<object> | 否 | 图片列表 |

示例：

```json
{
  "listing_id": "661a0c6af8d56b3a1a4b2024",
  "asset_mode": "centralized",
  "rent_mode": "whole",
  "city": "深圳",
  "district": "南山",
  "biz_area": "科技园",
  "community_name": "科技园公寓",
  "building_or_community_name": "A栋",
  "subway_station": "高新园",
  "subway_distance_m": 650,
  "title": "科技园公寓 A栋 1208",
  "subtitle": "2室1厅 | 48㎡ | 南向 | 12层",
  "price": 4200,
  "price_text": "4200元/月",
  "layout_text": "2室1厅",
  "area_size": 48,
  "orientation": "south",
  "floor_text": "12层",
  "payment_cycle": "monthly",
  "feature_flags": ["subway", "elevator"],
  "listing_facilities": ["subway", "elevator"],
  "platform_tags": ["近地铁", "拎包入住"],
  "images": [
    {
      "url": "https://example.com/room.jpg",
      "tag": "bedroom"
    }
  ]
}
```

### 2.2 `MiniappListingDetail`

详情对象在 `MiniappListingItem` 基础上增加：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `address_text` | string | 否 | 地址文案 |
| `geo` | object | 否 | 坐标 `{ lng, lat }` |
| `start_rent_rule` | string | 否 | 起租要求 |
| `cost_items` | array<object> | 否 | 费用明细 |
| `description` | string | 否 | 房源描述 |
| `risk_notice` | string | 否 | 风险提示 |
| `contact_phone` | string | 否 | 对外联系电话 |

### 2.3 `UserProfile`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `user_id` | string | 是 | 用户 ID，关联 `hs_usr_user._id` |
| `nickname` | string | 否 | 昵称 |
| `avatar` | string | 否 | 头像 URL |
| `phone` | string | 否 | 手机号 |
| `city` | string | 否 | 当前城市 |
| `budget_min` | int | 否 | 预算下限 |
| `budget_max` | int | 否 | 预算上限 |
| `preferred_areas` | array<string> | 否 | 偏好区域 |
| `preferred_rent_mode` | string | 否 | 偏好租住方式 |
| `move_in_plan` | string | 否 | 入住计划 |
| `remark` | string | 否 | 补充备注 |

## 3. 认证

### 3.1 微信登录

```text
POST /api/v1/auth/wechat_login
```

用途：微信已注册用户登录。未注册用户返回业务错误，前端应引导微信手机号授权注册。

鉴权：公开。

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `code` | string | 是 | `wx.login()` 返回的临时登录凭证 |

```json
{
  "code": "wx-login-code"
}
```

响应：

```json
{
  "code": 0,
  "error": "",
  "data": {
    "token": "opaque-bearer-token"
  }
}
```

### 3.2 微信注册

```text
POST /api/v1/auth/wechat_register
```

用途：微信新用户注册，并绑定微信授权手机号。注册成功后直接返回登录 token。

鉴权：公开。

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `code` | string | 是 | `wx.login()` 返回的临时登录凭证 |
| `phone_code` | string | 是 | `wx.getPhoneNumber()` 返回的手机号授权 code |

```json
{
  "code": "wx-login-code",
  "phone_code": "wx-phone-code"
}
```

响应：

```json
{
  "code": 0,
  "error": "",
  "data": {
    "token": "opaque-bearer-token"
  }
}
```

## 4. 用户

### 4.1 获取当前用户资料

```text
POST /api/v1/user/profile
```

用途：个人页展示用户基础信息和找房偏好。

鉴权：需要 Bearer token。

请求：空对象。

```json
{}
```

响应：

```json
{
  "code": 0,
  "error": "",
  "data": {
    "user_id": "661a0c6af8d56b3a1a4b2001",
    "nickname": "小明",
    "avatar": "https://example.com/avatar.jpg",
    "phone": "13800138000",
    "city": "深圳",
    "budget_min": 3000,
    "budget_max": 6000,
    "preferred_areas": ["南山", "福田"],
    "preferred_rent_mode": "whole",
    "move_in_plan": "one_month",
    "remark": ""
  }
}
```

### 4.2 更新当前用户资料

```text
POST /api/v1/user/update_profile
```

用途：编辑个人资料和找房偏好。

鉴权：需要 Bearer token。

语义：部分更新。未出现在请求体中的字段保持原值；传入空字符串或空数组表示主动清空该字段。

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `nickname` | string | 否 | 昵称，最长 50 |
| `avatar` | string | 否 | 头像 URL |
| `city` | string | 否 | 当前城市 |
| `budget_min` | int | 否 | 预算下限 |
| `budget_max` | int | 否 | 预算上限 |
| `preferred_areas` | array<string> | 否 | 偏好区域 |
| `preferred_rent_mode` | string | 否 | `whole` / `shared` |
| `move_in_plan` | string | 否 | 入住计划 |
| `remark` | string | 否 | 补充备注 |

```json
{
  "nickname": "小明",
  "avatar": "https://example.com/avatar.jpg",
  "city": "深圳",
  "budget_min": 3000,
  "budget_max": 6000,
  "preferred_areas": ["南山", "福田"],
  "preferred_rent_mode": "whole",
  "move_in_plan": "one_month",
  "remark": ""
}
```

响应：返回最新 `UserProfile`。

### 4.3 用户首页统计

```text
POST /api/v1/user/dashboard
```

用途：个人页首页统计。

鉴权：需要 Bearer token。

请求：空对象。

```json
{}
```

响应：

```json
{
  "code": 0,
  "error": "",
  "data": {
    "favorite_count": 3,
    "history_count": 12,
    "plan_count": 0,
    "unread_notification_count": 0
  }
}
```

字段说明：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `favorite_count` | int | 收藏房源数，只统计仍在线可展示房源 |
| `history_count` | int | 浏览足迹数，只统计仍在线可展示房源 |
| `plan_count` | int | 看房计划数 |
| `unread_notification_count` | int | 未读通知数 |

## 5. 找房

### 5.1 搜索在线房源

```text
POST /api/v1/house/search
```

用途：小程序找房列表。

鉴权：公开。

读取：只读 `hs_hpd_miniapp_listing`，不直接读取 HMD。

可见性：只返回 `is_online=1` 的 HPD 展示快照。

默认排序：`weight_score` 倒序，`updated_at` 倒序。

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `city` | string | 否 | 城市；为空时后端可使用当前产品默认城市 |
| `district` | string | 否 | 区域 |
| `biz_area` | string | 否 | 商圈 |
| `rent_mode` | string | 否 | `whole` / `shared` |
| `asset_mode` | string | 否 | `centralized` / `decentralized` |
| `min_price` | int | 否 | 最低租金 |
| `max_price` | int | 否 | 最高租金 |
| `keyword` | string | 否 | 匹配标题、项目/小区名、楼栋/小区名、地址、地铁站 |
| `feature_flags` | array<string> | 否 | 结构化筛选标记 |
| `page` | int | 否 | 默认 1 |
| `page_size` | int | 否 | 默认 20，最大 50 |

```json
{
  "city": "深圳",
  "district": "南山",
  "biz_area": "科技园",
  "rent_mode": "whole",
  "asset_mode": "centralized",
  "min_price": 3000,
  "max_price": 6000,
  "keyword": "高新园",
  "feature_flags": ["subway", "elevator"],
  "page": 1,
  "page_size": 20
}
```

响应：

```json
{
  "code": 0,
  "error": "",
  "data": {
    "list": [
      {
        "listing_id": "661a0c6af8d56b3a1a4b2024",
        "asset_mode": "centralized",
        "rent_mode": "whole",
        "city": "深圳",
        "district": "南山",
        "biz_area": "科技园",
        "community_name": "科技园公寓",
        "building_or_community_name": "A栋",
        "subway_station": "高新园",
        "subway_distance_m": 650,
        "title": "科技园公寓 A栋 1208",
        "subtitle": "2室1厅 | 48㎡ | 南向 | 12层",
        "price": 4200,
        "price_text": "4200元/月",
        "layout_text": "2室1厅",
        "area_size": 48,
        "orientation": "south",
        "floor_text": "12层",
        "payment_cycle": "monthly",
        "feature_flags": ["subway", "elevator"],
        "listing_facilities": ["subway", "elevator"],
        "platform_tags": ["近地铁", "拎包入住"],
        "images": [
          {
            "url": "https://example.com/room.jpg",
            "tag": "bedroom"
          }
        ]
      }
    ],
    "page": 1,
    "page_size": 20,
    "total": 1
  }
}
```

### 5.2 获取在线房源详情

```text
POST /api/v1/house/public_detail
```

用途：房源详情页。

鉴权：公开；带有效 token 时返回真实 `is_favorited`。无 token、无效 token、过期 token 均按匿名处理，返回 `is_favorited=false`。

读取：只读 `hs_hpd_miniapp_listing`。

可见性：非在线房源按未找到处理。

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `listing_id` | string | 是 | HPD 统一房源 ID |

```json
{
  "listing_id": "661a0c6af8d56b3a1a4b2024"
}
```

响应：

```json
{
  "code": 0,
  "error": "",
  "data": {
    "house": {
      "listing_id": "661a0c6af8d56b3a1a4b2024",
      "asset_mode": "centralized",
      "rent_mode": "whole",
      "city": "深圳",
      "district": "南山",
      "biz_area": "科技园",
      "community_name": "科技园公寓",
      "building_or_community_name": "A栋",
      "subway_station": "高新园",
      "subway_distance_m": 650,
      "address_text": "南山区科技园科发路",
      "geo": {
        "lng": 113.93452,
        "lat": 22.54012
      },
      "title": "科技园公寓 A栋 1208",
      "subtitle": "2室1厅 | 48㎡ | 南向 | 12层",
      "price": 4200,
      "price_text": "4200元/月",
      "layout_text": "2室1厅",
      "area_size": 48,
      "orientation": "south",
      "floor_text": "12层",
      "payment_cycle": "monthly",
      "feature_flags": ["subway", "elevator"],
      "listing_facilities": ["subway", "elevator"],
      "platform_tags": ["近地铁", "拎包入住"],
      "start_rent_rule": "long_one_year",
      "cost_items": [
        {
          "name": "押金",
          "amount": 4200,
          "unit": "元",
          "remark": "押一付一"
        }
      ],
      "description": "采光好，近地铁，适合通勤。",
      "risk_notice": "请通过平台确认看房与付款信息。",
      "contact_phone": "13800138000",
      "images": [
        {
          "url": "https://example.com/room.jpg",
          "tag": "bedroom"
        }
      ]
    },
    "is_favorited": false
  }
}
```

## 6. 收藏

### 6.1 添加收藏

```text
POST /api/v1/favorite/add
```

用途：收藏房源。

鉴权：需要 Bearer token。

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `listing_id` | string | 是 | HPD 统一房源 ID |

```json
{
  "listing_id": "661a0c6af8d56b3a1a4b2024"
}
```

响应：

```json
{
  "code": 0,
  "error": "",
  "data": {
    "listing_id": "661a0c6af8d56b3a1a4b2024",
    "is_favorited": true
  }
}
```

幂等：重复收藏同一 `listing_id` 返回成功。

### 6.2 取消收藏

```text
POST /api/v1/favorite/remove
```

用途：取消收藏房源。

鉴权：需要 Bearer token。

请求：

```json
{
  "listing_id": "661a0c6af8d56b3a1a4b2024"
}
```

响应：

```json
{
  "code": 0,
  "error": "",
  "data": {
    "listing_id": "661a0c6af8d56b3a1a4b2024",
    "is_favorited": false
  }
}
```

幂等：未收藏时取消收藏返回成功。

### 6.3 收藏列表

```text
POST /api/v1/favorite/list
```

用途：个人收藏列表。

鉴权：需要 Bearer token。

请求：

```json
{
  "page": 1,
  "page_size": 20
}
```

响应：

```json
{
  "code": 0,
  "error": "",
  "data": {
    "list": [
      {
        "listing_id": "661a0c6af8d56b3a1a4b2024",
        "asset_mode": "centralized",
        "rent_mode": "whole",
        "city": "深圳",
        "district": "南山",
        "biz_area": "科技园",
        "community_name": "科技园公寓",
        "title": "科技园公寓 A栋 1208",
        "subtitle": "2室1厅 | 48㎡ | 南向 | 12层",
        "price": 4200,
        "price_text": "4200元/月",
        "images": []
      }
    ],
    "page": 1,
    "page_size": 20,
    "total": 1
  }
}
```

说明：列表只返回仍在线的房源；已下架房源不在小程序收藏列表展示。`total` 与 `list` 口径一致，只统计仍在线可展示房源。

## 7. 足迹

### 7.1 添加浏览记录

```text
POST /api/v1/history/add
```

用途：进入房源详情页后记录浏览行为。

鉴权：需要 Bearer token。

请求：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `listing_id` | string | 是 | HPD 统一房源 ID |
| `source` | string | 是 | `discover` 列表；`ai` AI 推荐；`detail` 详情相关推荐 |

```json
{
  "listing_id": "661a0c6af8d56b3a1a4b2024",
  "source": "discover"
}
```

响应：

```json
{
  "code": 0,
  "error": "",
  "data": {
    "listing_id": "661a0c6af8d56b3a1a4b2024",
    "viewed_at": 1775451571
  }
}
```

同一用户重复浏览同一房源时，更新最近浏览时间。

### 7.2 浏览记录列表

```text
POST /api/v1/history/list
```

用途：个人足迹列表。

鉴权：需要 Bearer token。

请求：

```json
{
  "page": 1,
  "page_size": 20
}
```

响应：

```json
{
  "code": 0,
  "error": "",
  "data": {
    "list": [
      {
        "listing_id": "661a0c6af8d56b3a1a4b2024",
        "asset_mode": "centralized",
        "rent_mode": "whole",
        "city": "深圳",
        "district": "南山",
        "biz_area": "科技园",
        "community_name": "科技园公寓",
        "title": "科技园公寓 A栋 1208",
        "subtitle": "2室1厅 | 48㎡ | 南向 | 12层",
        "price": 4200,
        "price_text": "4200元/月",
        "images": [],
        "viewed_at": 1775451571
      }
    ],
    "page": 1,
    "page_size": 20,
    "total": 1
  }
}
```

说明：足迹按 `viewed_at` 倒序；只返回仍在线的房源。`total` 与 `list` 口径一致，只统计仍在线可展示房源。

## 8. 字段枚举

### 8.1 `asset_mode`

| 值 | 说明 |
| --- | --- |
| `centralized` | 集中式 |
| `decentralized` | 分散式 |

### 8.2 `rent_mode`

| 值 | 说明 |
| --- | --- |
| `whole` | 整租 |
| `shared` | 合租 |

### 8.3 `orientation`

| 值 | 说明 |
| --- | --- |
| `east` | 东 |
| `south` | 南 |
| `west` | 西 |
| `north` | 北 |
| `southeast` | 东南 |
| `northeast` | 东北 |
| `southwest` | 西南 |
| `northwest` | 西北 |
| `south_north` | 南北 |
| `unknown` | 未知 |

### 8.4 `payment_cycle`

| 值 | 说明 |
| --- | --- |
| `monthly` | 月付 |
| `quarterly` | 季付 |
| `half_yearly` | 半年付 |
| `yearly` | 年付 |
| `custom` | 自定义 |

### 8.5 `start_rent_rule`

| 值 | 说明 |
| --- | --- |
| `long_half_year` | 半年起租 |
| `long_one_year` | 一年起租 |
| `short_one_month` | 一个月起租 |
| `short_three_month` | 三个月起租 |
| `daily` | 日租 |

### 8.6 `feature_flags`

`feature_flags` 是小程序第一期结构化筛选字段。

适用位置：

- `POST /api/v1/house/search` 请求字段 `feature_flags`
- `MiniappListingItem.feature_flags`
- `MiniappListingDetail.feature_flags`

前端只能提交下表中的值。后端查询语义为：请求中每一个 `feature_flags` 值都必须在房源快照的 `feature_flags` 中存在。

| 值 | 前端展示文案 | 后端匹配语义 | 说明 |
| --- | --- | --- | --- |
| `subway` | 近地铁 | 房源 `feature_flags` 包含 `subway` | 用于原 `near_subway` 筛选 |
| `elevator` | 有电梯 | 房源 `feature_flags` 包含 `elevator` | 用于原 `has_elevator` 筛选 |
| `pet_friendly` | 可养宠 | 房源 `feature_flags` 包含 `pet_friendly` | 用于原 `pet_friendly` 筛选 |
| `cooking_allowed` | 可做饭 | 房源 `feature_flags` 包含 `cooking_allowed` | 用于原 `cooking_allowed` 筛选 |

约束：

- 未出现在本表中的值，前端不得提交。
- 后端收到未知值应返回参数错误。
- `feature_flags` 只表达结构化筛选，不替代 `platform_tags` 的运营展示标签。
