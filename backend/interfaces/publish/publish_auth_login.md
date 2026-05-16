# POST /api/v1/publish_auth/login

## 1. 入口

- 鉴权：公开接口。
- handler：`internal/handler/v1/publish/auth.(*PublicHandler).Login`
- service：`internal/service/publish/auth.(*Service).Login`

## 2. 请求体

```json
{
  "phone": "18002584637",
  "password": "123456"
}
```

## 3. 处理逻辑

1. handler 绑定 `phone/password`。
2. service 校验两个字段非空。
3. 通过 `repository/landlord.LandlordRepository.FindActiveByPhone` 查询 `hs_lld_landlord`。
4. 通过 `repository/landlord.LandlordAuthRepository.FindActivePasswordByLandlordID` 查询 `hs_lld_auth`。
5. 用 `bcrypt.CompareHashAndPassword` 校验密码。
6. 组装 `session.Principal`：
   - `terminal=publish`
   - `principal_type=landlord`
   - `principal_id=hs_lld_landlord._id`
   - `phone=hs_lld_landlord.phone`
7. 写入 Redis session，返回 token。
8. 尝试更新 `hs_lld_auth.last_login_at / last_login_ip`。

## 4. 数据读写

读取：

- `hs_lld_landlord`
- `hs_lld_auth`
- Redis session store（创建 token）

写入：

- Redis session token
- `hs_lld_auth.last_login_at`
- `hs_lld_auth.last_login_ip`

## 5. 失败返回

- `phone/password` 缺失：`InvalidParam`
- 房东不存在：`Unauthorized`
- 密码不匹配：`Unauthorized`
- Mongo 查询失败：`DatabaseError`
- Redis 写 session 失败：`CacheError`

## 6. 关键约束

- 当前 publish 登录固定是“房东手机号 + 密码”。
- Redis 只存登录态，不存账号密码真值。
- `role_codes / permission_codes` 当前固定返回空数组。

