# Backend Contract Review Checklist

用于后端按共享契约逐条确认，避免前后端联调漂移。

## 使用方式

- 每项标记：`[ ]` 未确认、`[x]` 已确认。
- 若不一致：记录“实际行为 + 计划修复时间”。

## A. 全局协议

- [x] 所有成功响应统一为 `{ "code": 0, "message": "ok", "data": ... }`。
- [x] 所有失败响应在 HTTP 状态码层表达，并可读取 `detail`。
- [ ] `detail` 当前多数仍为字符串，尚未统一为 `{ "code": "...", "message": "...", "details": {} }`。
- [x] 除登录/公开接口外，其余接口统一校验 `Authorization: Bearer <token>`。

## B. 认证接口

### `POST /api/v1/auth/login`
- [ ] 开发联调用接口可用。
- [ ] 请求体：`{ "user_id": "..." }`。
- [ ] 成功返回包含 `data.token`。

### `POST /api/v1/auth/wechat_login`
- [ ] 请求体：`{ "code": "wx-login-code" }`。
- [ ] 成功返回包含 `data.token`。
- [ ] token 可用于后续受保护接口。

## C. 聊天接口

### `POST /api/v1/chat/send`
- [ ] 请求体：`{ "message": "..." }`。
- [ ] 成功返回 `data.user` 与 `data.ai`。
- [ ] `data.ai.sentences` 为数组（前端按分句渲染）。
- [ ] 无 token 时返回未授权错误。

## D. 房源接口

### `POST /api/v1/house/search`（公开）
- [ ] 无 token 也可访问（若产品要求公开）。
- [ ] 支持参数：`area/min_price/max_price/type/tags/keyword/page/limit`。
- [ ] 返回包含：`data.total/page/limit/list`。
- [ ] 空值语义一致：`""`、`[]`、`0` 的行为与文档一致。
- [x] 租客可见房源返回已包含新版字段：`rent_mode/room_count/hall_count/bathroom_count/payment_cycle/near_subway/...`。

### `POST /api/v1/house/public_detail`
- [ ] 请求体：`{ "house_id": "..." }`。
- [ ] 返回包含：`data.house`、`data.is_favorited`。

### `POST /api/v1/house/create`
- [ ] 必填字段校验与文档一致。
- [ ] 仅房东/有权限用户可创建（按实际权限模型确认）。
- [ ] 当前仍沿用旧写入字段 `type`，尚未切到新版房源写模型。

### `POST /api/v1/house/list`
- [ ] 支持按 `status` 查询。
- [ ] `status` 枚举口径已固定（必须统一：`0/1/2` 或包含 `-1`）。

### `POST /api/v1/house/detail`
- [ ] 请求体：`{ "house_id": "..." }`。
- [ ] 仅可查看有权限的数据。

### `POST /api/v1/house/update`
- [ ] 请求体至少包含 `house_id`。
- [ ] 字段更新白名单清晰且稳定。
- [ ] 当前仍沿用旧写入字段 `type`，尚未切到新版房源写模型。

### `POST /api/v1/house/status`
- [ ] 请求体：`{ "house_id": "...", "status": ... }`。
- [ ] `status` 取值与 `house/list` 一致。

### `POST /api/v1/house/delete`
- [ ] 请求体：`{ "house_id": "..." }`。
- [ ] 删除策略口径明确（逻辑删/状态变更/物理删）。

## E. 收藏与历史

### `POST /api/v1/favorite/add`
- [ ] 请求体：`{ "house_id": "..." }`。
- [ ] 重复收藏行为定义清晰（幂等或报错）。

### `POST /api/v1/favorite/remove`
- [ ] 请求体：`{ "house_id": "..." }`。
- [ ] 未收藏时行为定义清晰（幂等或报错）。

### `POST /api/v1/favorite/list`
- [ ] 请求体支持 `limit`。
- [ ] 返回包含 `data.favorites`。

### `POST /api/v1/history/add`
- [ ] 请求体：`{ "house_id": "...", "source": "detail|discover|ai" }`。
- [ ] 可重复记录行为符合预期。

### `POST /api/v1/history/list`
- [ ] 请求体支持 `limit`。
- [ ] 返回包含 `data.history`。

## F. 用户接口

### `POST /api/v1/user/profile`
- [ ] 返回结构已定版（确认是 `data.profile` + `data.auth`，还是扁平 `data`）。
- [ ] `role` 字段语义清晰：当前生效角色。

### `POST /api/v1/user/dashboard`
- [ ] 租客字段：`favorite_count/history_count/session_count`。
- [ ] 房东字段：`house_total/house_on_rent/house_rented/house_offline`。
- [ ] 统计口径已与数据库实现对齐。

### `POST /api/v1/user/switch_role`
- [ ] 请求体：`{ "role": "tenant|landlord" }`。
- [ ] 成功后返回切换后的角色（建议必须返回）。

## G. 一致性检查

- [x] `shared-docs/api/openapi.yaml` 与后端实际路由一致。
- [x] 字段定义与 `shared-docs/schema/db-design/**/*.md` 已同步到当前租客侧实现。
- [ ] 页面调用链与 `shared-docs/api/frontend-endpoint-map.md` 一致。
- [ ] 若发现偏差，已同步更新文档或排期修复接口。
