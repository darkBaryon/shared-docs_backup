# Admin API

后台管理端接口契约文档。

本文档的目标不是一次性写完全部字段，而是先把第一阶段接口收成可以直接进入实现的形态。

当前说明：

- 账号与身份 schema 已明确：发房方主体为 `hs_lld_landlord`，认证为 `hs_lld_auth`。
- `hs_lld_profile` 已保留 schema 位置，但当前阶段不进入实现。
- 上架审核单模型与审核状态流仍待定，审核相关接口暂不进入代码实现。

## 约束

- 管理后台 API 独立于 miniapp 和 publish。
- 路径统一使用 `POST /api/v1/{module}/{action}`。
- 不直接复用 publish 端接口作为 admin 端接口。
- 接口动作围绕后台业务动作命名，而不是围绕底层集合命名。
- `auth` 因 miniapp 已占用 `/auth/session`，当前保留为 `admin_auth` 例外。
- 后台房源管理不再复用 miniapp 的 `house` 模块名，拆成 `house_root`、`house_building`、`house_room`，避免三端语义冲突。

## 第一阶段模块

第一阶段按实现顺序分为：

1. `admin_auth`
2. `staff`
3. `role`
4. `provider`
5. `house_root`
6. `house_building`
7. `house_room`
8. `launch_audit`

### `admin_auth`

- `POST /api/v1/admin_auth/login`
- `POST /api/v1/admin_auth/logout`
- `POST /api/v1/admin_auth/session`
- `POST /api/v1/admin_auth/reset_password`

详细后端处理逻辑按“一接口一文档”维护：

- [admin_auth/login](../backend/interfaces/admin/admin_auth_login.md)
- [admin_auth/session](../backend/interfaces/admin/admin_auth_session.md)
- [admin_auth/logout](../backend/interfaces/admin/admin_auth_logout.md)
- [admin_auth/reset_password](../backend/interfaces/admin/admin_auth_reset_password.md)

#### 鉴权约定

- 除 `admin_auth/login` 外，其余 `admin_auth` 接口都需要：

```text
Authorization: Bearer <token>
```

- token 是后端签发的 opaque Redis session token，不是 JWT。
- `admin_auth` 只生成 `terminal=admin`、`principal_type=staff` 的登录态。
- admin 业务接口不得提交 `staff_id` 作为登录主体，必须由后端从 session principal 推导。

#### 错误提示约定

- `error` 必须返回完整中文提示，不返回英文句子、枚举名或底层库报错。
- 参数错误返回可直接执行的中文提示，例如：`请输入手机号和密码`、`请求参数格式不正确`。
- 认证失败统一返回：`手机号或密码错误，请重新输入`，不暴露是手机号不存在、账号禁用还是密码错误。
- 登录态失效统一返回：`未登录或登录已过期，请重新登录` 或 `当前登录状态已失效，请重新登录`。
- 服务端内部异常统一返回克制的中文提示，例如：`登录失败，请稍后重试`、`获取登录状态失败，请稍后重试`、`退出登录失败，请稍后重试`。

#### `AdminPrincipal`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `principal_type` | string | 是 | 固定为 `staff` |
| `principal_id` | string | 是 | 当前员工 ObjectID hex |
| `terminal` | string | 是 | 固定为 `admin` |
| `phone` | string | 是 | 当前员工手机号 |
| `role_codes` | array<string> | 是 | 当前角色编码列表 |
| `permission_codes` | array<string> | 是 | 当前权限编码列表 |

#### `AdminStaffProfile`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `staff_id` | string | 是 | 员工 ID |
| `name` | string | 是 | 员工姓名 |
| `phone` | string | 是 | 员工手机号 |
| `email` | string | 否 | 员工邮箱 |
| `department` | string | 否 | 所属部门 |
| `job_title` | string | 否 | 岗位名称 |
| `contact_qr_code` | string | 否 | 联系二维码 URL |

#### `login`

请求：

- `phone`
- `password`

响应：

- `token`
- `principal`
  - `principal_type=staff`
  - `principal_id`
  - `phone`
- `role_codes`
- `permission_codes`

说明：

- `phone` 对应 `hs_adm_staff.phone`。
- `password` 对应 `hs_adm_staff_auth.password_hash` 的 bcrypt 校验。
- 登录失败统一返回“手机号或密码错误，请重新输入”，不暴露“手机号不存在”还是“密码错误”。

#### `session`

响应：

- `principal`
- `staff_profile`
- `role_codes`
- `permission_codes`

#### `logout`

响应：

- `success=true`

#### `reset_password`

请求：

- `phone`
- `captcha`
- `new_password`

说明：

- 第一阶段后移，不进入首批实现。

### `staff`

- `POST /api/v1/staff/list`
- `POST /api/v1/staff/detail`
- `POST /api/v1/staff/create`
- `POST /api/v1/staff/update`
- `POST /api/v1/staff/disable`

详细后端处理逻辑按“一接口一文档”维护：

- [staff/create](../backend/interfaces/admin/staff_create.md)
- [staff/list](../backend/interfaces/admin/staff_list.md)
- [staff/detail](../backend/interfaces/admin/staff_detail.md)
- [staff/update](../backend/interfaces/admin/staff_update.md)
- [staff/disable](../backend/interfaces/admin/staff_disable.md)

#### 权限约定

- `staff/list`、`staff/detail` 需要 `staff.view`。
- `staff/create`、`staff/update`、`staff/disable` 需要 `staff.edit`。
- 后端必须校验权限，前端菜单和按钮控制不能替代后端权限校验。

#### `list`

请求：

- `keyword`
- `phone`
- `role_id`
- `status`
- `page`
- `page_size`

说明：

- `status` 不传默认只查有效员工。
- 禁用员工通过 `status=-1` 显式筛选。
- `page` 默认 `1`，`page_size` 默认 `20`，最大 `100`。

响应：

- `list`
  - `staff_id`
  - `name`
  - `phone`
  - `email`
  - `department`
  - `job_title`
  - `contact_qr_code`
  - `roles`
    - `role_id`
    - `role_code`
    - `role_name`
  - `role_names`
  - `status`
  - `created_at`
  - `updated_at`
  - `last_login_at`
  - `last_login_ip`
- `page`
- `page_size`
- `total`

错误提示：

- 无权查看：`当前账号无权查看员工列表`
- 角色参数错误：`角色参数不正确`
- 员工状态参数错误：`员工状态参数不正确`
- 查询失败：`获取员工列表失败，请稍后重试`

#### `detail`

请求：

- `staff_id`

响应：

- `staff`
  - `staff_id`
  - `name`
  - `phone`
  - `email`
  - `department`
  - `job_title`
  - `contact_qr_code`
  - `roles`
    - `role_id`
    - `role_code`
    - `role_name`
  - `role_names`
  - `status`
  - `created_by_staff_id`
  - `created_at`
  - `updated_at`
  - `password_updated_at`
  - `last_login_at`
  - `last_login_ip`

错误提示：

- 无权查看：`当前账号无权查看员工详情`
- 员工参数错误：`员工参数不正确`
- 员工不存在：`员工不存在或已删除`
- 查询失败：`获取员工详情失败，请稍后重试`

#### `create`

请求：

- `name`
- `phone`
- `password`
- `email`
- `department`
- `job_title`
- `contact_qr_code`
- `role_ids`

说明：

- `name`、`phone`、`password` 必填。
- `phone` 全局唯一，创建后第一阶段不支持修改。
- `password` 只用于创建认证信息，响应不返回密码明文或 hash。
- `role_ids` 允许为空；无角色员工可以登录，但不能访问需要权限的业务功能。

响应：

- `staff`
  - `staff_id`
  - `name`
  - `phone`
  - `email`
  - `department`
  - `job_title`
  - `contact_qr_code`
  - `roles`
    - `role_id`
    - `role_code`
    - `role_name`
  - `role_names`
  - `status`
  - `created_at`
  - `updated_at`

错误提示：

- 无权创建：`当前账号无权创建员工`
- 必填缺失：`请输入员工姓名、手机号和初始密码`
- 手机号格式错误：`请输入正确的员工手机号`
- 手机号重复：`员工手机号已存在，请更换后重试`
- 角色无效：`所选角色不存在或已停用，请刷新后重试`
- 创建失败：`创建员工失败，请稍后重试`

#### `update`

请求：

- `staff_id`
- `name`
- `email`
- `department`
- `job_title`
- `contact_qr_code`
- `role_ids`
- `status`

说明：

- `staff_id` 必填。
- `phone` 不支持在该接口修改。
- `password` 不支持在该接口修改。
- `role_ids` 不传表示不调整角色；传空数组表示清空角色。
- 请求必须至少包含一个需要修改的字段。
- 调整角色或禁用员工后，后端会失效该员工已签发的 admin token。

响应：

- `staff`
  - 字段同 `staff/detail`

错误提示：

- 无权编辑：`当前账号无权编辑员工`
- 员工参数错误：`员工参数不正确`
- 没有提交修改字段：`请至少提交一个需要修改的字段`
- 员工姓名为空：`请输入员工姓名`
- 员工状态参数错误：`员工状态参数不正确`
- 角色参数错误：`角色参数不正确`
- 员工不存在：`员工不存在或已删除`
- 角色无效：`所选角色不存在或已停用，请刷新后重试`
- 更新失败：`更新员工失败，请稍后重试`

#### `disable`

请求：

- `staff_id`

响应：

- `success=true`

说明：

- 不允许禁用当前登录账号。
- 禁用后该员工旧 admin token 失效，不能继续访问后台业务接口。

错误提示：

- 无权禁用：`当前账号无权禁用员工`
- 员工参数错误：`员工参数不正确`
- 禁用当前登录账号：`不能禁用当前登录账号`
- 员工不存在：`员工不存在或已删除`
- 禁用失败：`禁用员工失败，请稍后重试`

### `role`

- `POST /api/v1/role/list`
- `POST /api/v1/role/detail`
- `POST /api/v1/role/create`
- `POST /api/v1/role/update`

详细后端处理逻辑按“一接口一文档”维护：

- [role/list](../backend/interfaces/admin/role_list.md)
- [role/detail](../backend/interfaces/admin/role_detail.md)
- [role/create](../backend/interfaces/admin/role_create.md)
- [role/update](../backend/interfaces/admin/role_update.md)

#### 权限约定

- `role/list`、`role/detail` 需要 `role.view`。
- `role/create`、`role/update` 需要 `role.edit`。

#### `list`

请求：

- `keyword`
- `page`
- `page_size`

响应：

- `list`
  - `role_id`
  - `role_name`
  - `role_code`
  - `description`
  - `permission_codes`
  - `permission_names`
  - `is_system`
  - `status`
  - `created_at`
  - `updated_at`
- `page`
- `page_size`
- `total`

#### `detail`

请求：

- `role_id`

响应：

- `role`
  - `role_id`
  - `role_name`
  - `role_code`
  - `description`
  - `permission_codes`
  - `permission_names`
  - `permissions`
  - `is_system`
  - `status`
  - `created_at`
  - `updated_at`

#### `create/update`

请求：

- `role_id`（仅 `update` 必填）
- `role_name`
- `role_code`（仅 `create` 必填，创建后不允许修改）
- `description`
- `permission_codes`

说明：

- `create` 时 `permission_codes` 必填且至少 1 个。
- `update` 时 `permission_codes` 不传表示不调整权限，传空数组不允许。
- 角色权限变更后，后端会失效已分配该角色员工的 admin token。

### `provider`

- `POST /api/v1/provider/list`
- `POST /api/v1/provider/detail`
- `POST /api/v1/provider/create`
- `POST /api/v1/provider/update`
- `POST /api/v1/provider/disable`

详细后端处理逻辑按“一接口一文档”维护：

- [provider/list](../backend/interfaces/admin/provider_list.md)
- [provider/detail](../backend/interfaces/admin/provider_detail.md)
- [provider/create](../backend/interfaces/admin/provider_create.md)
- [provider/update](../backend/interfaces/admin/provider_update.md)
- [provider/disable](../backend/interfaces/admin/provider_disable.md)

#### 权限约定

- `provider/list`、`provider/detail` 需要 `provider.view`。
- `provider/create`、`provider/update`、`provider/disable` 需要 `provider.edit`。

#### `list`

请求：

- `phone`
- `status`
- `page`
- `page_size`

响应：

- `list`
  - `provider_id`
  - `phone`
  - `status`
  - `created_by_staff_id`
  - `updated_by_staff_id`
  - `created_at`
  - `updated_at`
  - `password_updated_at`
  - `last_login_at`
  - `last_login_ip`
- `page`
- `page_size`
- `total`

#### `detail`

请求：

- `provider_id`

响应：

- `provider`
  - 字段同列表项

#### `create`

请求：

- `phone`
- `password`

#### `update`

请求：

- `provider_id`
- `phone`

#### `disable`

请求：

- `provider_id`

说明：

- `provider_id` 是接口层发房方 ID，正式等于 `hs_lld_landlord._id`。
- 当前阶段 provider 只实现房东账号主体与认证，即 `hs_lld_landlord + hs_lld_auth`。
- `name`、城市、图片、资质、品牌等资料依赖 `hs_lld_profile`，当前阶段不进入实现，不得塞进 `hs_lld_landlord` 或 `hs_usr_user`。
- 修改手机号或禁用发房方后，后端会失效该发房方旧 publish token。

### `house_root`

- `POST /api/v1/house_root/list`

#### `list`

请求：

- `provider_id`
- `asset_mode`
- `city`
- `district`
- `room_status`
- `listing_status`
- `audit_status`
- `page`
- `page_size`

响应：

- `list`
  - `root_id`
  - `root_type`
  - `root_name`
  - `asset_mode`
  - `provider_id`
  - `provider_phone`
  - `provider_name`
  - `project_id`
  - `project_name`
  - `community_id`
  - `community_name`
  - `city`
  - `district`
  - `biz_area`
  - `building_count`
  - `room_count`
  - `updated_at`

### `house_building`

- `POST /api/v1/house_building/list`

#### `list`

请求：

- `root_id`
- `room_status`
- `listing_status`
- `audit_status`
- `page`
- `page_size`

响应：

- `list`
  - `root_id`
  - `building_id`
  - `project_id`
  - `project_name`
  - `building_name`
  - `city`
  - `district`
  - `biz_area`
  - `room_count`
  - `updated_at`

### `house_room`

- `POST /api/v1/house_room/list`
- `POST /api/v1/house_room/detail`

#### `list`

请求：

- `root_id`
- `building_id`
- `provider_id`
- `asset_mode`
- `city`
- `district`
- `room_status`
- `listing_status`
- `audit_status`
- `page`
- `page_size`

响应：

- `list`
  - `listing_id`
  - `asset_mode`
  - `provider_id`
  - `provider_phone`
  - `title`
  - `city`
  - `district`
  - `biz_area`
  - `community_name`
  - `building_name`
  - `room_no`
  - `price`
  - `price_text`
  - `layout_text`
  - `area_size`
  - `room_status`
  - `listing_status`
  - `audit_status`
  - `is_online`
  - `updated_at`

#### `detail`

请求：

- `listing_id`

响应：

- `room`
  - 字段同列表项
  - `source_type`
  - `source_id`
  - `root_type`
  - `root_id`
  - `project_id`
  - `project_name`
  - `building_id`
  - `room_type_id`
  - `room_type_name`
  - `decentralized_id`
  - `address_text`
  - `latest_audit_task_id`
  - `latest_submitted_at`
  - `latest_reviewed_at`
  - `reviewer_staff_id`

说明：

- 后台房源管理首页改为项目/小区列表，不再直接平铺房间。
- 房间列表和详情读取 `hs_hpd_admin_listing`，该 collection 是管理后台房源总览 read model。
- `house_room/export` 不进入第一批实现，后续需要先补需求和接口文档。

### `launch_audit`

- `POST /api/v1/launch_audit/list`
- `POST /api/v1/launch_audit/detail`
- `POST /api/v1/launch_audit/approve`
- `POST /api/v1/launch_audit/reject`

#### `list`

请求：

- `source`
- `provider_id`
- `asset_mode`
- `status`
- `page`
- `page_size`

#### `detail`

请求：

- `audit_id`

#### `approve`

请求：

- `audit_id`
- `remark`

#### `reject`

请求：

- `audit_id`
- `remark`

## 关联文档

- 总开发计划：`workstreams/admin-web/active/admin-mvp.md`
- 后端详细计划：`workstreams/admin-web/active/admin-backend-plan.md`
- 前端详细计划：`workstreams/admin-web/active/admin-frontend-plan.md`
