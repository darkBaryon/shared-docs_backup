# POST /api/v1/admin_auth/login

## 1. 当前状态

该接口进入 admin 第一阶段首批实现。

目标：

- 员工使用“手机号 + 密码”登录后台。
- 后端签发 `terminal=admin` 的 Redis session token。
- 响应返回当前 staff 主体摘要和角色权限快照。

## 2. 入口

- 鉴权：公开接口。
- handler：`internal/handler/v1/admin/auth.(*PublicHandler).Login`
- service：`internal/service/admin/auth.(*Service).Login`

## 3. 请求体

```json
{
  "phone": "13800000000",
  "password": "123456"
}
```

字段说明：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `phone` | string | 是 | 后台员工手机号，对应 `hs_adm_staff.phone` |
| `password` | string | 是 | 员工登录密码，使用 bcrypt hash 校验 |

## 4. 响应体

```json
{
  "token": "opaque-token",
  "principal": {
    "principal_type": "staff",
    "principal_id": "staff_object_id",
    "terminal": "admin",
    "phone": "13800000000",
    "role_codes": ["super_admin"],
    "permission_codes": ["staff.view", "staff.edit"]
  },
  "staff_profile": {
    "staff_id": "staff_object_id",
    "name": "管理员",
    "phone": "13800000000",
    "email": "",
    "department": "",
    "job_title": "",
    "contact_qr_code": ""
  },
  "role_codes": ["super_admin"],
  "permission_codes": ["staff.view", "staff.edit"]
}
```

说明：

- `principal.role_codes` 与响应顶层 `role_codes` 保持一致，方便 middleware 和前端分别读取。
- `principal.permission_codes` 与响应顶层 `permission_codes` 保持一致。
- `staff_profile` 只返回员工资料，不返回密码 hash。

## 5. 处理逻辑

1. handler 绑定 `phone/password`。
2. service 校验 `phone/password` 非空。
3. 通过 `repository/staff.StaffRepository.FindActiveByPhone` 查询 `hs_adm_staff`。
4. 如果员工不存在或 `status != active`，返回 `Unauthorized`，错误文案统一为“手机号或密码错误，请重新输入”。
5. 通过 `repository/staff.StaffAuthRepository.FindActivePasswordByStaffID` 查询 `hs_adm_staff_auth`。
6. 用 `bcrypt.CompareHashAndPassword` 校验密码。
7. 查询员工角色：
   - `hs_adm_staff_role`
   - `hs_adm_role`
8. 查询角色权限：
   - `hs_adm_role_permission`
   - `hs_adm_permission`
9. 组装 `session.Principal`：
   - `terminal=admin`
   - `principal_type=staff`
   - `principal_id=hs_adm_staff._id`
   - `phone=hs_adm_staff.phone`
   - `role_codes=[]string`
   - `permission_codes=[]string`
10. 写入 Redis session，返回 token。
11. 更新 `hs_adm_staff_auth.last_login_at / last_login_ip`。
12. 写入 `hs_adm_login_log` 成功日志。

## 6. 数据读写

读取：

- `hs_adm_staff`
- `hs_adm_staff_auth`
- `hs_adm_staff_role`
- `hs_adm_role`
- `hs_adm_role_permission`
- `hs_adm_permission`

写入：

- Redis session token
- `hs_adm_staff_auth.last_login_at`
- `hs_adm_staff_auth.last_login_ip`
- `hs_adm_login_log`

## 7. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| `phone/password` 缺失 | `InvalidParam` | `请输入手机号和密码` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| 员工不存在 | `Unauthorized` | `手机号或密码错误，请重新输入` |
| 员工已禁用 | `Unauthorized` | `手机号或密码错误，请重新输入` |
| 员工认证记录不存在或被禁用 | `Unauthorized` | `手机号或密码错误，请重新输入` |
| 密码不匹配 | `Unauthorized` | `手机号或密码错误，请重新输入` |
| 角色或权限查询失败 | `DatabaseError` | `登录失败，请稍后重试` |
| Redis 写 session 失败 | `CacheError` | `登录失败，请稍后重试` |
| 登录审计日志写入失败 | 无业务失败 | 不阻断登录，只记录服务端业务日志 |

## 8. 关键约束

- admin 登录主体固定为 `staff`。
- admin session 固定为 `terminal=admin`、`principal_type=staff`。
- 登录手机号来自 `hs_adm_staff.phone`，密码 hash 来自 `hs_adm_staff_auth.password_hash`。
- 不复用 `publish_auth`，不读取 `hs_lld_landlord / hs_lld_auth`。
- 登录失败不能泄露“手机号不存在”或“密码错误”的具体差异，统一返回“手机号或密码错误，请重新输入”。
- 第一阶段只支持密码登录，不支持验证码、邮箱登录、多因子登录。
