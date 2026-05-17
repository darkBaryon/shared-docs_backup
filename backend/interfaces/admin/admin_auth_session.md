# POST /api/v1/admin_auth/session

## 1. 当前状态

该接口进入 admin 第一阶段首批实现。

目标：

- 校验当前 Bearer token 是否仍然有效。
- 基于当前 staff 主体重新计算角色权限。
- 返回当前 admin 后台登录态。

## 2. 入口

- 鉴权：需要 Bearer token。
- handler：`internal/handler/v1/admin/auth.(*Handler).Session`
- service：`internal/service/admin/auth.(*Service).Session`

## 3. 请求体

空对象即可：

```json
{}
```

## 4. 响应体

```json
{
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

## 5. 处理逻辑

1. auth middleware 从 Redis 读取 token，并把 `session.Principal` 写入 context。
2. handler 从 context 读取 principal。
3. service 校验：
   - `terminal == admin`
   - `principal_type == staff`
4. 通过 `principal_id` 回查 `hs_adm_staff`。
5. 如果员工不存在或已禁用，返回 `Unauthorized`，错误文案统一为“当前登录状态已失效，请重新登录”。
6. 重新查询角色和权限：
   - `hs_adm_staff_role`
   - `hs_adm_role`
   - `hs_adm_role_permission`
   - `hs_adm_permission`
7. 重新组装当前登录态并返回。

## 6. 数据读写

读取：

- Redis session store
- `hs_adm_staff`
- `hs_adm_staff_role`
- `hs_adm_role`
- `hs_adm_role_permission`
- `hs_adm_permission`

写入：

- 无业务表写入
- Redis session 在 auth middleware 读取成功后按统一 session 策略滑动续期

## 7. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| token 缺失 | `Unauthorized` | `未登录或登录已过期，请重新登录` |
| token 无效或已过期 | `Unauthorized` | `未登录或登录已过期，请重新登录` |
| principal 不是 `admin + staff` | `Unauthorized` | `当前登录状态无效，请重新登录` |
| `principal_id` 不是合法 ObjectID | `Unauthorized` | `当前登录状态无效，请重新登录` |
| 员工不存在或已禁用 | `Unauthorized` | `当前登录状态已失效，请重新登录` |
| 角色或权限查询失败 | `DatabaseError` | `获取登录状态失败，请稍后重试` |

## 8. 关键约束

- session 恢复必须回查员工主体，不能只信 Redis 里的旧 principal。
- session 恢复必须重新计算角色权限，避免员工角色调整后旧权限长期有效。
- admin session 不接受 `miniapp + user` 或 `publish + landlord` principal。
