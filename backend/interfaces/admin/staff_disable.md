# POST /api/v1/staff/disable

## 1. 当前状态

该接口进入 admin Phase 1 `staff` 模块实现，用于禁用后台员工账号。

目标：

- 后台有权限员工禁用指定后台员工。
- 禁用采用软状态更新，不物理删除员工。
- 禁用后该员工不能再次登录。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `staff.edit`。
- handler：`internal/handler/v1/admin/staff.(*Handler).Disable`
- service：`internal/service/admin/staff.(*Service).Disable`

## 3. 请求体

```json
{
  "staff_id": "staff_object_id"
}
```

字段说明：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `staff_id` | string | 是 | 员工 ObjectID hex |

## 4. 响应体

```json
{
  "success": true
}
```

## 5. 处理逻辑

1. auth middleware 校验 admin Bearer token，并注入 `session.Principal`。
2. 权限 middleware 校验当前 principal 包含 `staff.edit`。
3. handler 绑定 `staff_id`。
4. service 校验 `staff_id` 是合法 ObjectID。
5. 查询 `hs_adm_staff`：
   - 允许命中 `status=1` 有效员工。
   - 允许命中 `status=-1` 已禁用员工，重复禁用按成功处理。
6. 如果 `staff_id` 等于当前登录员工 ID，返回 `InvalidParam`。
7. 如果员工不存在，返回 `NotFound`。
8. 将 `hs_adm_staff.status` 更新为 `-1`。
9. 后端失效该员工已签发的 admin session。
10. 返回 `success=true`。

## 6. 数据读写

读取：

- Redis session store
- `hs_adm_staff`

写入：

- `hs_adm_staff`
- Redis session version

## 7. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| token 缺失 | `Unauthorized` | `未登录或登录已过期，请重新登录` |
| token 无效或已过期 | `Unauthorized` | `未登录或登录已过期，请重新登录` |
| principal 不是 `admin + staff` | `Unauthorized` | `当前登录状态无效，请重新登录` |
| 缺少 `staff.edit` 权限 | `Forbidden` | `当前账号无权禁用员工` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| `staff_id` 缺失或格式错误 | `InvalidParam` | `员工参数不正确` |
| 禁用当前登录账号 | `InvalidParam` | `不能禁用当前登录账号` |
| 员工不存在 | `NotFound` | `员工不存在或已删除` |
| 更新员工失败 | `DatabaseError` | `禁用员工失败，请稍后重试` |

## 8. 关键约束

- 禁用员工只更新 `hs_adm_staff.status=-1`，不物理删除员工。
- 禁用员工不删除历史角色关系，便于恢复时保留原角色。
- 操作人来自 admin session，不从请求体读取当前登录员工 ID。
- 不允许禁用当前登录账号。
- 禁用后新登录会被拒绝，旧 admin token 也必须失效。
