# POST /api/v1/admin_auth/logout

## 1. 当前状态

该接口进入 admin 第一阶段首批实现。

目标：

- 清理当前 Bearer token 对应的 Redis session。
- 只退出当前登录态，不影响其它 token。

## 2. 入口

- 鉴权：需要 Bearer token。
- handler：`internal/handler/v1/admin/auth.(*Handler).Logout`
- service：`internal/service/admin/auth.(*Service).Logout`

## 3. 请求体

空对象即可：

```json
{}
```

## 4. 响应体

```json
{
  "success": true
}
```

## 5. 处理逻辑

1. auth middleware 从请求头解析 Bearer token。
2. handler 从 context 读取当前 token。
3. service 调用 `session.Store.Delete(token)` 删除 Redis session。
4. 返回 `success=true`。

## 6. 数据读写

读取：

- Redis session store

写入：

- 删除 Redis session token

## 7. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| token 缺失 | `Unauthorized` | `未登录或登录已过期，请重新登录` |
| token 无效或已过期 | `Unauthorized` | `未登录或登录已过期，请重新登录` |
| Redis 删除失败 | `CacheError` | `退出登录失败，请稍后重试` |

## 8. 关键约束

- logout 只清理当前 token，不修改员工主体、角色、权限。
- logout 不写 `hs_adm_login_log`，登录审计只记录登录行为。
