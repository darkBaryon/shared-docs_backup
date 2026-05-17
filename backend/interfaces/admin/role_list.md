# POST /api/v1/role/list

## 1. 当前状态

该接口进入 admin Phase 1 `role` 模块实现。

目标：

- 后台员工查看角色列表。
- 支持按角色名称或角色编码关键词筛选。
- 返回角色基础信息和权限摘要。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `role.view`。
- handler：`internal/handler/v1/admin/role.(*Handler).List`
- service：`internal/service/admin/role.(*Service).List`

## 3. 请求体

```json
{
  "keyword": "运营",
  "page": 1,
  "page_size": 20
}
```

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `keyword` | string | 否 | 角色名称或角色编码关键词 |
| `page` | int | 否 | 页码，默认 `1` |
| `page_size` | int | 否 | 每页数量，默认 `20`，最大 `100` |

## 4. 响应体

```json
{
  "list": [
    {
      "role_id": "role_object_id",
      "role_name": "运营管理员",
      "role_code": "ops_admin",
      "description": "运营管理角色",
      "permission_codes": ["staff.view"],
      "permission_names": ["查看员工"],
      "is_system": 0,
      "status": 1,
      "created_at": 1760000000,
      "updated_at": 1760000000
    }
  ],
  "page": 1,
  "page_size": 20,
  "total": 1
}
```

## 5. 处理逻辑

1. auth middleware 校验 admin Bearer token，并注入 `session.Principal`。
2. 权限 middleware 校验当前 principal 包含 `role.view`。
3. handler 绑定请求体，空 body 按 `{}` 处理。
4. service 规范化分页。
5. 查询 `hs_adm_role` 有效角色。
6. 批量查询 `hs_adm_role_permission` 和 `hs_adm_permission`。
7. 返回角色列表。

## 6. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| 缺少 `role.view` 权限 | `Forbidden` | `当前账号无权查看角色列表` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| 查询失败 | `DatabaseError` | `获取角色列表失败，请稍后重试` |

## 7. 关键约束

- 列表只返回有效角色。
- 权限码来自后端权限字典，不接受前端自造权限码。
