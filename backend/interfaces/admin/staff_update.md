# POST /api/v1/staff/update

## 1. 当前状态

该接口进入 admin Phase 1 `staff` 模块实现，用于维护后台员工资料和角色。

目标：

- 后台有权限员工更新员工基础资料。
- 支持替换员工角色。
- 不支持修改员工手机号和密码。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `staff.edit`。
- handler：`internal/handler/v1/admin/staff.(*Handler).Update`
- service：`internal/service/admin/staff.(*Service).Update`

## 3. 请求体

```json
{
  "staff_id": "staff_object_id",
  "name": "运营一号",
  "email": "ops@example.com",
  "department": "运营部",
  "job_title": "运营",
  "contact_qr_code": "https://example.com/qrcode.png",
  "status": 1,
  "role_ids": ["role_object_id"]
}
```

字段说明：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `staff_id` | string | 是 | 员工 ObjectID hex |
| `name` | string | 否 | 员工姓名；传入时去首尾空格后不能为空 |
| `email` | string | 否 | 员工邮箱；可传空字符串清空 |
| `department` | string | 否 | 所属部门；可传空字符串清空 |
| `job_title` | string | 否 | 岗位名称；可传空字符串清空 |
| `contact_qr_code` | string | 否 | 对外联系二维码 URL；可传空字符串清空 |
| `status` | int | 否 | 员工状态；只接受 `1` 或 `-1` |
| `role_ids` | array<string> | 否 | 角色 ObjectID hex 列表；传空数组表示清空角色；不传表示不调整角色 |

说明：

- `phone` 不在该接口中修改。
- `password` 不在该接口中修改。
- 请求必须至少包含一个需要修改的字段。

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

- 响应结构与 `staff/detail` 一致。
- 不返回 `password` 或 `password_hash`。

## 5. 处理逻辑

1. auth middleware 校验 admin Bearer token，并注入 `session.Principal`。
2. 权限 middleware 校验当前 principal 包含 `staff.edit`。
3. handler 绑定请求体。
4. service 校验 `staff_id` 是合法 ObjectID。
5. service 校验至少提交一个可修改字段。
6. 查询 `hs_adm_staff`：
   - 允许更新 `status=1` 有效员工。
   - 允许更新 `status=-1` 禁用员工。
7. 如果员工不存在，返回 `NotFound`。
8. 规范化并校验字段：
   - `name` 传入时去首尾空格后不能为空。
   - `status` 只接受 `1` 或 `-1`。
   - `role_ids` 传入时去重，并校验每一项都是合法 ObjectID。
9. 如果传入 `role_ids`，查询 `hs_adm_role`，确认所有角色存在且有效。
10. 更新 `hs_adm_staff` 的基础字段。
11. 如果传入 `role_ids`，替换 `hs_adm_staff_role` 中该员工的有效角色关系。
12. 如果调整了角色，或将员工状态更新为 `-1`，后端失效该员工已签发的 admin session。
13. 查询最新详情并返回。

## 6. 数据读写

读取：

- Redis session store
- `hs_adm_staff`
- `hs_adm_staff_auth`
- `hs_adm_staff_role`
- `hs_adm_role`

写入：

- `hs_adm_staff`
- `hs_adm_staff_role`

## 7. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| token 缺失 | `Unauthorized` | `未登录或登录已过期，请重新登录` |
| token 无效或已过期 | `Unauthorized` | `未登录或登录已过期，请重新登录` |
| principal 不是 `admin + staff` | `Unauthorized` | `当前登录状态无效，请重新登录` |
| 缺少 `staff.edit` 权限 | `Forbidden` | `当前账号无权编辑员工` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| `staff_id` 缺失或格式错误 | `InvalidParam` | `员工参数不正确` |
| 没有提交修改字段 | `InvalidParam` | `请至少提交一个需要修改的字段` |
| `name` 为空 | `InvalidParam` | `请输入员工姓名` |
| `status` 不支持 | `InvalidParam` | `员工状态参数不正确` |
| `role_ids` 格式错误 | `InvalidParam` | `角色参数不正确` |
| 员工不存在 | `NotFound` | `员工不存在或已删除` |
| 角色不存在或已禁用 | `InvalidParam` | `所选角色不存在或已停用，请刷新后重试` |
| 更新员工失败 | `DatabaseError` | `更新员工失败，请稍后重试` |
| 更新角色失败 | `DatabaseError` | `更新员工失败，请稍后重试` |

## 8. 关键约束

- 该接口不修改员工手机号。
- 该接口不修改员工密码。
- 角色只能通过 `role_ids` 绑定有效角色，不能直接提交角色名或权限码落库。
- 操作人来自 admin session，不从请求体读取当前登录员工 ID。
- 权限必须由后端校验，前端隐藏按钮不能代替后端权限控制。
- 调整角色或禁用员工后，旧 admin token 必须失效，避免 Redis principal 中的旧权限继续生效。
