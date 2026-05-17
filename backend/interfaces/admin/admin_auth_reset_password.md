# POST /api/v1/admin_auth/reset_password

## 1. 当前状态

该接口保留在 API 契约中，但**不进入首批后端实现**。

首批 admin auth 只实现：

- `login`
- `session`
- `logout`

## 2. 预留接口定位

当前只做契约占位，不生成 handler / service 实现。

该接口后续若启动实现，仍归属 `admin_auth` 模块，避免把员工自助改密、管理员代改密码、验证码找回密码混在同一接口里。

## 3. 入口

- 鉴权：待定。
- handler：`internal/handler/v1/admin/auth.(*PublicHandler).ResetPassword` 或 `internal/handler/v1/admin/auth.(*Handler).ResetPassword`
- service：`internal/service/admin/auth.(*Service).ResetPassword`

## 4. 预期请求体

```json
{
  "phone": "13800000000",
  "captcha": "123456",
  "new_password": "new-password"
}
```

字段说明：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `phone` | string | 是 | 员工手机号 |
| `captcha` | string | 是 | 验证码 |
| `new_password` | string | 是 | 新密码 |

## 5. 暂不实现原因

- 当前系统尚未设计短信/邮箱验证码能力。
- 当前 admin 首批目标是“员工可登录后台”，不是完整找回密码链路。
- 员工初始密码与重置密码第一阶段优先由 `staff/create` 或后续 `staff/reset_password` 这类后台受保护接口完成。

## 6. 后续实现方向

可选方向：

1. 公开找回密码：
   - `phone + captcha + new_password`
   - 需要验证码发送、校验、频控、防刷
2. 管理员重置员工密码：
   - 放入 `POST /api/v1/staff/reset_password`
   - 需要 `staff.edit` 权限

当前建议：

- 第一阶段不实现公开 `reset_password`。
- 如果必须做改密，优先做受保护的 `staff/reset_password`，由管理员操作。

错误提示预期：

- 参数问题使用明确中文提示，例如：`请输入手机号`、`请输入验证码`、`请输入新密码`。
- 验证失败不返回内部判断细节，统一使用可执行中文提示，例如：`验证码无效或已过期`。
