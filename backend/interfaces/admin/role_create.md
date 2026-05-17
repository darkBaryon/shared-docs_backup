# POST /api/v1/role/create

## 1. 当前状态

该接口进入 admin Phase 1 `role` 模块实现。

目标：

- 后台有权限员工创建角色。
- 创建角色时分配权限。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `role.edit`。
- handler：`internal/handler/v1/admin/role.(*Handler).Create`
- service：`internal/service/admin/role.(*Service).Create`

## 3. 请求体

```json
{
  "role_name": "运营管理员",
  "role_code": "ops_admin",
  "description": "运营管理角色",
  "permission_codes": ["staff.view", "staff.edit"]
}
```

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role_name` | string | 是 | 角色名称 |
| `role_code` | string | 是 | 角色稳定编码，创建后不可修改 |
| `description` | string | 否 | 角色说明 |
| `permission_codes` | array<string> | 是 | 权限编码列表，至少 1 个 |

## 4. 响应体

```json
{
  "role": {
    "role_id": "role_object_id",
    "role_name": "运营管理员",
    "role_code": "ops_admin",
    "description": "运营管理角色",
    "permission_codes": ["staff.view", "staff.edit"],
    "permission_names": ["查看员工", "编辑员工"],
    "is_system": 0,
    "status": 1,
    "created_at": 1760000000,
    "updated_at": 1760000000
  }
}
```

## 5. 处理逻辑

1. 权限 middleware 校验当前 principal 包含 `role.edit`。
2. service 校验 `role_name`、`role_code`、`permission_codes`。
3. 查询 `hs_adm_role`，确认 `role_code` 未被占用。
4. 查询 `hs_adm_permission`，确认所有权限编码存在且有效。
5. 写入 `hs_adm_role`。
6. 写入 `hs_adm_role_permission`。
7. 返回角色摘要。

非事务约定：

- 当前开发阶段不启用 Mongo 多集合事务。
- 写入角色后，如果权限关系写入失败，后端按本次创建的 `role_id` 清理角色和权限关系。

## 6. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| 缺少 `role.edit` 权限 | `Forbidden` | `当前账号无权创建角色` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| 必填缺失 | `InvalidParam` | `请输入角色名称、角色编码和权限` |
| 权限为空 | `InvalidParam` | `请至少选择一个权限` |
| 权限无效 | `InvalidParam` | `所选权限不存在或已停用，请刷新后重试` |
| 角色编码重复 | `AlreadyExists` | `角色编码已存在，请更换后重试` |
| 创建失败 | `DatabaseError` | `创建角色失败，请稍后重试` |

## 7. 关键约束

- `role_code` 是稳定业务编码，创建后不允许修改。
- 权限只能使用后端权限字典中的有效 `permission_code`。
