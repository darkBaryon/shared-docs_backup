# Backend Auth

## 当前链路

- 小程序微信登录 / 注册由 `handler/v1/miniapp/auth` 和 `service/miniapp/auth` 处理。
- 微信身份绑定、用户初始化、资料扩展档兜底等逻辑收敛在 `service/miniapp/auth` 内部。
- 登录成功后写入 Redis session。
- 客户端通过 `Authorization: Bearer <token>` 访问受保护接口。
- middleware 从 Redis 读取 session，并把用户信息写入 Gin context。

约束：

- 小程序 auth 是端侧应用服务，不作为 publish/admin 的通用登录入口。
- 后续 publish/admin auth 应分别建立自己的 `service/{terminal}/auth`。

## Session Principal

Redis session token 是 opaque token。客户端只能透传 token，不能解析 token，也不能从 token 自行推导权限。

session value 必须保存结构化 principal JSON，而不是只保存字符串 `user_id`：

```json
{
  "principal_type": "staff",
  "principal_id": "ObjectID hex",
  "terminal": "publish",
  "phone": "13800000000",
  "role_codes": ["super_admin"],
  "permission_codes": ["house.manage"]
}
```

字段约束：

| 字段 | 说明 |
| --- | --- |
| `principal_type` | 身份类型，当前为 `user` 或 `staff` |
| `principal_id` | 当前身份在账号域的 ObjectID hex |
| `terminal` | 当前登录终端，当前为 `miniapp`、`publish` 或 `admin` |
| `phone` | 当前身份手机号，用于 owner phone 数据作用域匹配 |
| `role_codes` | 当前身份角色编码，不存在时为空数组 |
| `permission_codes` | 当前身份权限编码，不存在时为空数组 |

## 终端边界

- `miniapp`：小程序登录入口，只产生 `terminal=miniapp` 的 session；默认 `principal_type=user`。
- `publish`：出房 Web 登录入口使用 `POST /api/v1/publish_auth/login`、`session`、`logout`；MVP 优先产生 `terminal=publish`、`principal_type=staff` 的 session。
- `admin`：后台管理登录入口独立设计，必须产生 `terminal=admin` 的 session。

约束：

- publish/admin 不复用小程序微信 login 作为通用登录入口。
- publish 业务 middleware 必须校验 `terminal=publish`，不能只校验 token 是否存在。
- miniapp、publish、admin 可以复用同一个 Redis session store，但必须通过 principal 的 `terminal` 和 `principal_type` 区分边界。
- 前端业务请求不得提交 `user_id`、`staff_id` 或 owner 过滤字段；后端从 principal 推导数据作用域。
- HMD 不增加 owner/user/staff 归属字段；房源归属和服务关系属于 HPD entrust relation。

## 相关文档

- [../api/auth-flow.md](../api/auth-flow.md)
- [../api/publish.md](../api/publish.md)
- [../schema/db-design/v4/modules/account-and-identity.md](../schema/db-design/v4/modules/account-and-identity.md)
