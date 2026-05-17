# POST /api/v1/staff/create

## 1. 当前状态

该接口进入 admin Phase 1 `staff` 模块首批实现，并优先于 `staff/list` 实现。

目标：

- 后台有权限员工创建新的后台员工账号。
- 同时写入员工主体、密码认证信息和员工角色关系。
- 返回新员工基础资料和角色摘要。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `staff.edit`。
- handler：`internal/handler/v1/admin/staff.(*Handler).Create`
- service：`internal/service/admin/staff.(*Service).Create`

## 3. 请求体

```json
{
  "name": "运营一号",
  "phone": "13800000000",
  "password": "123456",
  "email": "ops@example.com",
  "department": "运营部",
  "job_title": "运营",
  "contact_qr_code": "https://example.com/qrcode.png",
  "role_ids": ["role_object_id"]
}
```

字段说明：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `name` | string | 是 | 员工姓名，去首尾空格后不能为空 |
| `phone` | string | 是 | 员工手机号，全局唯一 |
| `password` | string | 是 | 初始密码，后端使用 bcrypt hash 后写入 |
| `email` | string | 否 | 员工邮箱 |
| `department` | string | 否 | 所属部门 |
| `job_title` | string | 否 | 岗位名称 |
| `contact_qr_code` | string | 否 | 对外联系二维码 URL |
| `role_ids` | array<string> | 否 | 角色 ObjectID hex 列表；允许为空 |

说明：

- `role_ids` 允许为空；无角色员工可以登录，但不能访问需要权限的业务功能。
- `phone` 第一阶段创建后不支持修改。
- 当前阶段不支持由员工创建接口发送短信或邮件通知。

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
    "created_at": 1760000000,
    "updated_at": 1760000000
  }
}
```

说明：

- 响应不返回 `password` 或 `password_hash`。
- 创建员工不会自动为新员工签发 session。

## 5. 处理逻辑

1. auth middleware 校验 admin Bearer token，并注入 `session.Principal`。
2. 权限 middleware 校验当前 principal 包含 `staff.edit`。
3. handler 绑定请求体。
4. service 校验并规范化输入：
   - `name` 去首尾空格后不能为空。
   - `phone` 去首尾空格后不能为空，且格式为手机号。
   - `password` 去首尾空格后不能为空。
   - `role_ids` 去重，并校验每一项都是合法 ObjectID。
5. 查询 `hs_adm_staff`，确认手机号未被有效员工占用。
6. 如果传入 `role_ids`，查询 `hs_adm_role`，确认所有角色存在且有效。
7. 使用 bcrypt 生成密码 hash。
8. 写入 `hs_adm_staff`：
   - `name`
   - `phone`
   - `email`
   - `department`
   - `job_title`
   - `contact_qr_code`
   - `created_by_staff_id`
   - `status=1`
9. 写入 `hs_adm_staff_auth`：
   - `staff_id`
   - `auth_type=password`
   - `password_hash`
   - `password_updated_at`
   - `status=1`
10. 写入 `hs_adm_staff_role`：
   - `staff_id`
   - `role_id`
   - `assigned_by`
   - `assigned_at`
11. 返回新员工资料和角色摘要。

非事务约定：

- 当前开发阶段不启用 Mongo 多集合事务。
- service 必须保证失败路径不返回“创建成功”的假结果。
- 写入 `hs_adm_staff` 后，如果认证信息或角色关系写入失败，后端按本次创建的 `staff_id` 清理已写入的员工、认证信息和角色关系。
- 生产环境后续如启用 Mongo transaction，再以事务替换显式补偿，不改变接口契约。

## 6. 数据读写

读取：

- Redis session store
- `hs_adm_staff`
- `hs_adm_role`

写入：

- `hs_adm_staff`
- `hs_adm_staff_auth`
- `hs_adm_staff_role`

## 7. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| token 缺失 | `Unauthorized` | `未登录或登录已过期，请重新登录` |
| token 无效或已过期 | `Unauthorized` | `未登录或登录已过期，请重新登录` |
| principal 不是 `admin + staff` | `Unauthorized` | `当前登录状态无效，请重新登录` |
| 缺少 `staff.edit` 权限 | `Forbidden` | `当前账号无权创建员工` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| `name/phone/password` 缺失 | `InvalidParam` | `请输入员工姓名、手机号和初始密码` |
| 手机号格式错误 | `InvalidParam` | `请输入正确的员工手机号` |
| `role_ids` 格式错误 | `InvalidParam` | `角色参数不正确` |
| 手机号已存在 | `AlreadyExists` | `员工手机号已存在，请更换后重试` |
| 角色不存在或已禁用 | `InvalidParam` | `所选角色不存在或已停用，请刷新后重试` |
| 写入员工失败 | `DatabaseError` | `创建员工失败，请稍后重试` |
| 写入认证信息失败 | `DatabaseError` | `创建员工失败，请稍后重试` |
| 写入角色关系失败 | `DatabaseError` | `创建员工失败，请稍后重试` |

## 8. 关键约束

- 该接口创建的是后台员工，不是 C 端用户，也不是发房方。
- 密码只允许以 bcrypt hash 写入 `hs_adm_staff_auth.password_hash`。
- 不返回密码明文或密码 hash。
- 操作人来自 admin session，不从请求体读取 `created_by_staff_id`。
- 创建员工不自动登录新员工，不签发新员工 token。
- `role_ids` 只能绑定有效角色；不能接受前端提交的任意角色名或权限码直接落库。
