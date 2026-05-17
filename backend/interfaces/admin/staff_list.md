# POST /api/v1/staff/list

## 1. 当前状态

该接口进入 admin Phase 1 `staff` 模块首批实现。

目标：

- 后台员工查看员工列表。
- 支持按姓名关键词、手机号、角色、状态筛选。
- 返回员工基础资料、角色摘要和最近登录信息。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `staff.view`。
- handler：`internal/handler/v1/admin/staff.(*Handler).List`
- service：`internal/service/admin/staff.(*Service).List`

## 3. 请求体

```json
{
  "keyword": "运营",
  "phone": "13800000000",
  "role_id": "role_object_id",
  "status": 1,
  "page": 1,
  "page_size": 20
}
```

字段说明：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `keyword` | string | 否 | 员工姓名关键词，去首尾空格后模糊匹配 |
| `phone` | string | 否 | 员工手机号，按完整手机号匹配 |
| `role_id` | string | 否 | 角色 ObjectID hex，筛选分配了该角色的员工 |
| `status` | int | 否 | 员工状态；`1` 有效，`-1` 禁用；不传默认只查 `1` |
| `page` | int | 否 | 页码，默认 `1` |
| `page_size` | int | 否 | 每页数量，默认 `20`，最大 `100` |

说明：

- `keyword` 与 `phone` 同时传入时取交集。
- `role_id` 只接受合法 ObjectID hex。
- 当前阶段不提供按邮箱、部门、岗位的独立筛选。

## 4. 响应体

```json
{
  "list": [
    {
      "staff_id": "staff_object_id",
      "name": "运营一号",
      "phone": "13800000000",
      "email": "",
      "department": "运营部",
      "job_title": "运营",
      "contact_qr_code": "",
      "roles": [
        {
          "role_id": "role_object_id",
          "role_code": "ops_admin",
          "role_name": "运营管理员"
        }
      ],
      "role_names": ["运营管理员"],
      "status": 1,
      "created_at": 1760000000,
      "updated_at": 1760000000,
      "last_login_at": 1760000000,
      "last_login_ip": "127.0.0.1"
    }
  ],
  "page": 1,
  "page_size": 20,
  "total": 1
}
```

字段说明：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `staff_id` | string | 员工 ObjectID hex |
| `name` | string | 员工姓名 |
| `phone` | string | 员工手机号 |
| `email` | string | 员工邮箱 |
| `department` | string | 所属部门 |
| `job_title` | string | 岗位名称 |
| `contact_qr_code` | string | 对外联系二维码 URL |
| `roles` | array | 员工当前角色摘要 |
| `role_names` | array<string> | 角色名称冗余字段，方便列表展示 |
| `status` | int | 员工状态 |
| `created_at` | int64 | 创建时间，Unix 秒 |
| `updated_at` | int64 | 更新时间，Unix 秒 |
| `last_login_at` | int64 | 最近登录时间，来自 `hs_adm_staff_auth` |
| `last_login_ip` | string | 最近登录 IP，来自 `hs_adm_staff_auth` |

## 5. 处理逻辑

1. auth middleware 校验 admin Bearer token，并注入 `session.Principal`。
2. 权限 middleware 校验当前 principal 包含 `staff.view`。
3. handler 绑定请求体，空 body 按 `{}` 处理。
4. service 规范化分页：
   - `page <= 0` 时按 `1`。
   - `page_size <= 0` 时按 `20`。
   - `page_size > 100` 时按 `100`。
5. service 校验 `status`：
   - 未传默认 `1`。
   - 只接受 `1` 或 `-1`。
6. service 校验 `role_id`，非空时必须是合法 ObjectID。
7. 如果传入 `role_id`，先查 `hs_adm_staff_role` 得到员工 ID 集合；无匹配时直接返回空列表。
8. 查询 `hs_adm_staff`，按筛选条件分页返回员工主体。
9. 批量查询员工角色：
   - `hs_adm_staff_role`
   - `hs_adm_role`
10. 批量查询员工认证摘要：
   - `hs_adm_staff_auth.last_login_at`
   - `hs_adm_staff_auth.last_login_ip`
11. 组装列表响应。

排序规则：

- 默认按 `created_at desc`。
- 同一创建时间按 `_id desc` 保持稳定顺序。

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
| 缺少 `staff.view` 权限 | `Forbidden` | `当前账号无权查看员工列表` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| `role_id` 格式错误 | `InvalidParam` | `角色参数不正确` |
| `status` 不支持 | `InvalidParam` | `员工状态参数不正确` |
| 查询员工失败 | `DatabaseError` | `获取员工列表失败，请稍后重试` |
| 查询角色失败 | `DatabaseError` | `获取员工列表失败，请稍后重试` |

## 8. 关键约束

- 该接口只读，不修改员工、角色或登录信息。
- 列表不得返回 `password_hash` 或任何认证密钥。
- 操作人来自 admin session，不从请求体读取 `staff_id`。
- 默认只查有效员工；禁用员工必须通过 `status=-1` 显式筛选。
- 权限必须由后端校验，前端隐藏菜单不能代替后端权限控制。
