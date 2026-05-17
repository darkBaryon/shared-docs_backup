# POST /api/v1/role/update

## 1. 当前状态

该接口进入 admin Phase 1 `role` 模块实现。

目标：

- 后台有权限员工更新角色名称、说明和权限。
- 角色权限变更后，已分配该角色员工的旧 admin token 失效。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `role.edit`。
- handler：`internal/handler/v1/admin/role.(*Handler).Update`
- service：`internal/service/admin/role.(*Service).Update`

## 3. 请求体

```json
{
  "role_id": "role_object_id",
  "role_name": "运营管理员",
  "description": "运营管理角色",
  "permission_codes": ["staff.view", "staff.edit"]
}
```

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role_id` | string | 是 | 角色 ObjectID hex |
| `role_name` | string | 否 | 角色名称，传入时不能为空 |
| `description` | string | 否 | 角色说明，可传空字符串清空 |
| `permission_codes` | array<string> | 否 | 权限编码列表；不传表示不调整权限，传空数组不允许 |

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
2. service 校验 `role_id` 和至少提交一个可修改字段。
3. 查询 `hs_adm_role`，确认角色存在且有效。
4. 如果传入 `permission_codes`，查询 `hs_adm_permission`，确认所有权限存在且有效。
5. 更新 `hs_adm_role` 基础字段。
6. 如果传入 `permission_codes`，替换 `hs_adm_role_permission` 有效关系。
7. 如果调整了权限，失效已分配该角色员工的 admin session。
8. 返回最新角色摘要。

## 6. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| 缺少 `role.edit` 权限 | `Forbidden` | `当前账号无权编辑角色` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| `role_id` 缺失或格式错误 | `InvalidParam` | `角色参数不正确` |
| 没有提交修改字段 | `InvalidParam` | `请至少提交一个需要修改的字段` |
| 角色名称为空 | `InvalidParam` | `请输入角色名称` |
| 权限为空 | `InvalidParam` | `请至少选择一个权限` |
| 权限无效 | `InvalidParam` | `所选权限不存在或已停用，请刷新后重试` |
| 角色不存在 | `NotFound` | `角色不存在或已停用` |
| 更新失败 | `DatabaseError` | `更新角色失败，请稍后重试` |

## 7. 关键约束

- `role_code` 不支持更新。
- 权限变更后旧 admin token 必须失效，避免旧权限继续生效。
