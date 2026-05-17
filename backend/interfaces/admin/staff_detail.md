# POST /api/v1/staff/detail

## 1. 当前状态

该接口进入 admin Phase 1 `staff` 模块实现，排在 `staff/create`、`staff/list` 之后。

目标：

- 后台员工查看单个员工详情。
- 返回员工基础资料、角色摘要、创建人和登录摘要。
- 支持查看有效员工和已禁用员工。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `staff.view`。
- handler：`internal/handler/v1/admin/staff.(*Handler).Detail`
- service：`internal/service/admin/staff.(*Service).Detail`

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
  "staff": {
    "staff_id": "staff_object_id",
    "name": "运营一号",
    "phone": "13800000000",
    "email": "ops@example.com",
    "department": "运营部",
    "job_title": "运营",
    "contact_qr_code": "https://example.com/qrcode.png",
    "roles": [
      {
        "role_id": "role_object_id",
        "role_code": "ops_admin",
        "role_name": "运营管理员"
      }
    ],
    "role_names": ["运营管理员"],
    "status": 1,
    "created_by_staff_id": "creator_staff_object_id",
    "created_at": 1760000000,
    "updated_at": 1760000000,
    "password_updated_at": 1760000000,
    "last_login_at": 1760000000,
    "last_login_ip": "127.0.0.1"
  }
}
```

说明：

- 不返回 `password` 或 `password_hash`。
- `created_by_staff_id` 为空字符串表示系统初始化或历史种子数据。
- `password_updated_at`、`last_login_at` 为 `0` 表示暂无记录。

## 5. 处理逻辑

1. auth middleware 校验 admin Bearer token，并注入 `session.Principal`。
2. 权限 middleware 校验当前 principal 包含 `staff.view`。
3. handler 绑定 `staff_id`。
4. service 校验 `staff_id` 是合法 ObjectID。
5. 查询 `hs_adm_staff`：
   - 允许返回 `status=1` 有效员工。
   - 允许返回 `status=-1` 禁用员工。
6. 如果员工不存在，返回 `NotFound`。
7. 查询员工角色：
   - `hs_adm_staff_role`
   - `hs_adm_role`
8. 查询员工认证摘要：
   - `hs_adm_staff_auth.password_updated_at`
   - `hs_adm_staff_auth.last_login_at`
   - `hs_adm_staff_auth.last_login_ip`
9. 组装详情响应。

## 6. 数据读写

读取：

- Redis session store
- `hs_adm_staff`
- `hs_adm_staff_auth`
- `hs_adm_staff_role`
- `hs_adm_role`

写入：

- 无业务写入。

## 7. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| token 缺失 | `Unauthorized` | `未登录或登录已过期，请重新登录` |
| token 无效或已过期 | `Unauthorized` | `未登录或登录已过期，请重新登录` |
| principal 不是 `admin + staff` | `Unauthorized` | `当前登录状态无效，请重新登录` |
| 缺少 `staff.view` 权限 | `Forbidden` | `当前账号无权查看员工详情` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| `staff_id` 缺失或格式错误 | `InvalidParam` | `员工参数不正确` |
| 员工不存在 | `NotFound` | `员工不存在或已删除` |
| 查询员工失败 | `DatabaseError` | `获取员工详情失败，请稍后重试` |
| 查询角色失败 | `DatabaseError` | `获取员工详情失败，请稍后重试` |
| 查询认证摘要失败 | `DatabaseError` | `获取员工详情失败，请稍后重试` |

## 8. 关键约束

- 该接口只读，不修改员工、角色或登录信息。
- 详情不得返回 `password_hash` 或任何认证密钥。
- 操作人来自 admin session，不从请求体读取当前登录员工 ID。
- 权限必须由后端校验，前端隐藏按钮不能代替后端权限控制。
