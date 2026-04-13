# Backend Sync Changelog

给前端联调用的本轮后端变更记录。

## 2026-04-13

### 全局协议

- 成功响应统一为 `{ code: 0, message: "ok", data: ... }`。
- HTTP 2xx 只代表请求成功到达后端，不代表业务一定成功。
- 失败时目前多数仍返回字符串 `detail`，前端不要假设 `detail` 一定是对象。

### 用户与认证

- `wechat_login` 成功后，`data.token` 可直接用于后续受保护接口。
- 接口里出现的 `user_id`，现在实际是用户 Mongo `_id` 的字符串形式，不再是单独的业务 ID。

### 房源

- 租客侧公开房源返回已补充新版字段：
  - `rent_mode`
  - `room_count`
  - `hall_count`
  - `bathroom_count`
  - `payment_cycle`
  - `near_subway`
  - `walk_to_subway_min`
  - `pet_friendly`
  - `has_elevator`
  - `cooking_allowed`
  - `furnished_level`
- `type` 目前仍保留，用于兼容前端展示，它是由 `room_count + hall_count` 派生出来的展示字段。
- `POST /api/v1/house/public_detail` 现在返回：
  - `data.house`
  - `data.is_favorited`

### 收藏与历史

- 已支持：
  - `POST /api/v1/favorite/add`
  - `POST /api/v1/favorite/remove`
  - `POST /api/v1/favorite/list`
  - `POST /api/v1/history/add`
  - `POST /api/v1/history/list`
- 房源详情页可直接用 `is_favorited` 控制收藏按钮状态。
- 进入房源详情页时，可以直接调用 `history/add` 记录浏览行为。

### 当前范围说明

- 当前已经对齐的是租客侧主链路：登录、搜房、详情、收藏、历史、用户主页、AI 找房。
- 房东侧 `house/create`、`house/update` 还没有完全切到新版写模型，后续会单独调整。
