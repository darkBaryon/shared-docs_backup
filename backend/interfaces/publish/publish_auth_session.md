# POST /api/v1/publish_auth/session

## 1. 入口

- 鉴权：需要 Bearer token。
- handler：`internal/handler/v1/publish/auth.(*Handler).Session`
- service：`internal/service/publish/auth.(*Service).Session`

## 2. 处理逻辑

1. auth middleware 从 Redis 读取 token，对应 `session.Principal` 写入 context。
2. handler 从 context 读取 principal。
3. service 校验：
   - `terminal == publish`
   - `principal_type == landlord`
4. 通过 `principal_id` 回查 `hs_lld_landlord`。
5. 重新组装当前登录态并返回。

## 3. 数据读写

读取：

- Redis session store
- `hs_lld_landlord`

写入：

- 无

## 4. 失败返回

- token 缺失或无效：`Unauthorized`
- principal 不是 `publish + landlord`：`Unauthorized`
- Mongo 查询失败：`DatabaseError`
- principal 中的 `principal_id` 非法：`Unauthorized`

## 5. 关键约束

- session 恢复不查 staff，不查 miniapp user。
- publish 端 session 恢复只认房东 principal。

