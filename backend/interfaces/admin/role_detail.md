# POST /api/v1/role/detail

## 1. 当前状态

该接口进入 admin Phase 1 `role` 模块实现。

目标：

- 后台员工查看单个角色详情。
- 返回角色基础信息和完整权限列表。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `role.view`。
- handler：`internal/handler/v1/admin/role.(*Handler).Detail`
- service：`internal/service/admin/role.(*Service).Detail`

## 3. 请求体

```json
{
  "role_id": "role_object_id"
}
```

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role_id` | string | 是 | 角色 ObjectID hex |

## 4. 响应体

```json
{
  "role": {
    "role_id": "role_object_id",
    "role_name": "运营管理员",
    "role_code": "ops_admin",
    "description": "运营管理角色",
    "permission_codes": ["staff.view"],
    "permission_names": ["查看员工"],
    "permissions": [
      {
        "permission_id": "permission_object_id",
        "permission_code": "staff.view",
        "permission_name": "查看员工",
        "module": "staff",
        "action": "view"
      }
    ],
    "is_system": 0,
    "status": 1,
    "created_at": 1760000000,
    "updated_at": 1760000000
  }
}
```

## 5. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| 缺少 `role.view` 权限 | `Forbidden` | `当前账号无权查看角色详情` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| `role_id` 缺失或格式错误 | `InvalidParam` | `角色参数不正确` |
| 角色不存在 | `NotFound` | `角色不存在或已停用` |
| 查询失败 | `DatabaseError` | `获取角色详情失败，请稍后重试` |

## 6. 关键约束

- 详情只返回有效角色。
- 权限列表来自 `hs_adm_permission`。
