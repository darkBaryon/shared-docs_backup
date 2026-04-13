# 智小窝找房数据库完整结构设计（V2）

## 1. 设计目标

本版结构面向当前小程序实际需求：
- 租客侧：公开找房、咨询、收藏、浏览历史、个人主页统计
- 房东侧：房源管理、上下架、个人主页统计
- 账号侧：微信登录、角色切换、用户资料

统一原则：
- 主路径查询可被索引覆盖
- 行为数据与用户静态数据分离
- 统计字段优先“实时聚合”，不冗余写入 `user` 主表

---

## 2. 集合总览

1. `hs_usr_user`：用户主档案
2. `hs_usr_auth`：登录凭证映射（微信 openid 等）
3. `hs_hs_house`：房源详情
4. `hs_chat_session`：会话状态
5. `hs_chat_message`：聊天流水
6. `hs_usr_favorite`：租客收藏
7. `hs_usr_history`：租客浏览历史

---

## 3. 详细结构

## 3.1 `hs_usr_user`（用户主档案）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `user_id` | String | 是 | 业务唯一 ID |
| `role` | String | 是 | 当前角色：`tenant` / `landlord` |
| `nickname` | String | 否 | 用户昵称 |
| `avatar` | String | 否 | 头像 URL |
| `contact` | Object | 否 | 例：`{"wechat":"", "phone":""}` |
| `global_preferences` | Object | 是 | 用户全局偏好 |
| `status` | Int | 是 | `1` 正常 |
| `created_at` | Int64 | 是 | 创建时间 |
| `last_active` | Int64 | 是 | 最后活跃时间 |
| `updated_at` | Int64 | 是 | 更新时间 |

索引：
- `{ user_id: 1 }` unique
- `{ role: 1, status: 1 }`

---

## 3.2 `hs_usr_auth`（认证映射）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `_id` | String | 是 | 主键 |
| `user_id` | String | 是 | 关联 `hs_usr_user.user_id` |
| `identity_type` | String | 是 | `wechat` |
| `identifier` | String | 是 | 微信 `openid` |
| `credential` | String | 否 | 预留（密码/密钥） |
| `status` | Int | 是 | `1` 正常 |
| `created_at` | Int64 | 是 | 创建时间 |
| `updated_at` | Int64 | 是 | 更新时间 |

索引：
- `{ identity_type: 1, identifier: 1 }` unique
- `{ user_id: 1, status: 1 }`

---

## 3.3 `hs_hs_house`（房源详情，V5）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `_id` | String | 是 | 主键 |
| `owner_id` | String | 是 | 房东 ID |
| `area` | String | 是 | 行政区（南山/福田/宝安/罗湖） |
| `location` | String | 是 | 小区/街道 |
| `price` | Int | 是 | 月租金 |
| `status` | Int | 是 | `1`待租 / `2`已租 / `0`下架 |
| `images` | List[String] | 否 | 图片 URL 列表 |
| `rent_mode` | String | 是 | `whole` / `shared` |
| `room_count` | Int | 否 | 室数量 |
| `hall_count` | Int | 否 | 厅数量 |
| `bathroom_count` | Int | 否 | 卫数量 |
| `payment_cycle` | String | 否 | `押一付一` / `押一付三` / `月付` / `季付` |
| `near_subway` | Bool | 否 | 是否近地铁 |
| `walk_to_subway_min` | Int | 否 | 步行到地铁分钟数 |
| `pet_friendly` | Bool | 否 | 是否可养宠物 |
| `has_elevator` | Bool | 否 | 是否有电梯 |
| `cooking_allowed` | Bool | 否 | 是否可做饭 |
| `furnished_level` | String | 否 | `none` / `basic` / `full` |
| `tags` | List[String] | 否 | 展示标签（可由结构化字段生成） |
| `created_at` | Int64 | 是 | 创建时间 |
| `updated_at` | Int64 | 是 | 更新时间 |

索引：
- `{ owner_id: 1, status: 1 }`
- `{ status: 1, area: 1, price: 1 }`
- `{ status: 1, rent_mode: 1, room_count: 1, price: 1 }`
- `{ status: 1, near_subway: 1, walk_to_subway_min: 1 }`
- `{ status: 1, payment_cycle: 1, price: 1 }`
- `{ status: 1, tags: 1 }`（multikey）

---

## 3.4 `hs_chat_session`（会话状态）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `_id` | String | 是 | 会话 ID |
| `user_id` | String | 是 | 用户 ID |
| `state` | Int | 是 | 状态机阶段 |
| `requirement` | Object | 否 | 本次需求快照 |
| `off_topic_count` | Int | 是 | 离题计数 |
| `status` | Int | 是 | `1` 正常 |
| `created_at` | Int64 | 是 | 创建时间 |
| `updated_at` | Int64 | 是 | 更新时间 |

索引：
- `{ user_id: 1, updated_at: -1 }`

---

## 3.5 `hs_chat_message`（聊天流水）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `session_id` | String | 是 | 会话 ID |
| `role` | String | 是 | `user` / `assistant` |
| `content` | String | 是 | 消息内容 |
| `status` | Int | 是 | `1` 正常 |
| `timestamp` | Int64 | 是 | 时间戳 |

索引：
- `{ session_id: 1, timestamp: 1 }`

---

## 3.6 `hs_usr_favorite`（收藏）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `_id` | String | 是 | 主键 |
| `user_id` | String | 是 | 用户 ID |
| `house_id` | String | 是 | 房源 ID |
| `status` | Int | 是 | `1` 已收藏 / `0` 取消收藏 |
| `created_at` | Int64 | 是 | 创建时间 |
| `updated_at` | Int64 | 是 | 更新时间 |

索引：
- `{ user_id: 1, house_id: 1 }` unique
- `{ user_id: 1, status: 1, updated_at: -1 }`

---

## 3.7 `hs_usr_history`（浏览历史）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `_id` | String | 是 | 主键 |
| `user_id` | String | 是 | 用户 ID |
| `house_id` | String | 是 | 房源 ID |
| `source` | String | 否 | 来源：`discover` / `ai` / `detail` |
| `status` | Int | 是 | `1` 正常 |
| `viewed_at` | Int64 | 是 | 浏览时间 |
| `created_at` | Int64 | 是 | 创建时间 |
| `updated_at` | Int64 | 是 | 更新时间 |

索引：
- `{ user_id: 1, viewed_at: -1 }`
- `{ user_id: 1, house_id: 1, viewed_at: -1 }`

---

## 4. 统计口径（用于 `/api/v1/user/dashboard`）

租客：
- `favorite_count` = `hs_usr_favorite` 中 `user_id` 且 `status=1` 的数量
- `history_count` = `hs_usr_history` 中 `user_id` 且 `status=1` 的数量
- `session_count` = `hs_chat_session` 中 `user_id` 且 `status=1` 的数量

房东：
- `house_total` = `hs_hs_house` 中 `owner_id` 且 `status in (0,1,2)` 数量
- `house_on_rent` = `status=1`
- `house_rented` = `status=2`
- `house_offline` = `status=0`

说明：以上均建议实时聚合，不冗余写入 `hs_usr_user`。

---

## 5. 接口与集合映射

- `/api/v1/auth/wechat_login` -> `hs_usr_auth`, `hs_usr_user`
- `/api/v1/user/profile` -> `hs_usr_user`
- `/api/v1/user/switch_role` -> `hs_usr_user.role`
- `/api/v1/user/dashboard` -> 聚合 `favorite/history/session/house`
- `/api/v1/house/search` -> `hs_hs_house`
- `/api/v1/house/create|update|status|delete|list` -> `hs_hs_house`
- `/api/v1/favorite/add|remove|list` -> `hs_usr_favorite`
- `/api/v1/history/add|list` -> `hs_usr_history`
- `/api/v1/chat/send` -> `hs_chat_session`, `hs_chat_message`

---

