# POST /api/v1/publish_auth/logout

## 1. 入口

- 鉴权：需要 Bearer token。
- handler：`internal/handler/v1/publish/auth.(*Handler).Logout`
- service：`internal/service/publish/auth.(*Service).Logout`

## 2. 处理逻辑

1. handler 从 middleware context 读取当前 token。
2. service 校验 token 非空。
3. 调用 Redis session store 删除该 token。
4. 返回 `logged_out=true`。

## 3. 数据读写

读取：

- middleware context token

写入：

- Redis 删除 session token

## 4. 失败返回

- token 缺失：`Unauthorized`
- Redis 删除失败：`CacheError`

## 5. 关键约束

- logout 不改 Mongo 账号状态。
- logout 不做全端踢下线，只删当前 token。

