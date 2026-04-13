# User Schema

## UserProfile

- `user_id`
  - 类型：`string`
  - 含义：用户 `_id` 的字符串形式，也是当前前后端传递的用户标识。
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

## ID 约定

- MongoDB 主键统一使用 `_id: ObjectId`。
- 接口对外暴露 `user_id` 时，实际内容为用户 `_id` 的字符串形式。
- `auth.user_id`、`favorite.user_id`、`history.user_id`、`session.user_id` 等关联字段在数据库内部统一使用 `ObjectId`。
