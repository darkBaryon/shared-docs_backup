# User Schema

## UserProfile

- `user_id`
  - 类型：`string`
  - 含义：用户业务唯一标识。
- `role`
  - 类型：`string`
  - 枚举：`tenant` | `landlord`
  - 含义：当前生效角色。
- `nickname`
  - 类型：`string | null`
  - 含义：用户昵称。
- `avatar`
  - 类型：`string | null`
  - 含义：头像 URL。
- `contact`
  - 类型：`object | null`
  - 示例：`{"wechat":"", "phone":""}`
  - 含义：联系方式。
- `created_at`
  - 类型：`int64`（秒级时间戳）
  - 含义：注册或创建时间。
