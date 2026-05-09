# Backend Sync Changelog

给前端联调用的本轮后端变更记录。

## 2026-05-10

### Publish 发房端

- 发房端后端第一期 HMD 链路已通过真实服务联调。
- 可联调接口统一为 `POST /api/v1/publish/{action}`。
- 受保护接口需要 `Authorization: Bearer <session-token>`。
- 已验证对象：
  - 集中式项目
  - 楼栋
  - 集中式房型
  - 集中式房间
  - 分散式小区
  - 分散式房间
- 已验证动作：
  - 创建
  - 详情
  - 列表
  - 更新
  - 房态更新
- 当前 HPD 仍是 no-op，前端不要把 HMD 数据当成小程序展示层数据。
- 分散式房间当前没有房型模型，不提供 `room_type_id`。

## 2026-04-16

### AI Chat

- `POST /api/v1/chat/send` 响应结构升级为多句输出：
  - `data.ai.sentences: string[]`（前端按分句渲染）
- 共享契约已移除旧的单字符串 `data.ai.text`。
- 同时补充可观测字段（可选）：
  - `data.ai.model`
  - `data.ai.ai_tone_score`
  - `data.ai.quality_score`
  - `data.ai.raw_model_output`
- `data.user.text` / `data.user.timestamp`、`data.ai.timestamp` 保持不变。

## 2026-04-13

### 全局协议

- Go 后端成功响应统一为 `{ code: 0, error: "", data: ... }`。
- HTTP 2xx 只代表请求成功到达后端，不代表业务一定成功。
- 失败时优先读取 `error`。

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
