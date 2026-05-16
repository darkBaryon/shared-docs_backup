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
- `auth` 因 miniapp 已占用 `/auth/session`，当前保留为 `admin_auth` 例外；其它 admin 模块不加 terminal 前缀。

## 第一阶段模块

第一阶段按实现顺序分为：

1. `admin_auth`
2. `staff`
3. `role`
4. `provider`
5. `house`
6. `launch_audit`

### `admin_auth`

- `POST /api/v1/admin_auth/login`
- `POST /api/v1/admin_auth/logout`
- `POST /api/v1/admin_auth/session`
- `POST /api/v1/admin_auth/reset_password`

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

#### `list`

请求：

- `keyword`
- `phone`
- `role_id`
- `status`
- `page`
- `page_size`

响应：

- `list`
  - `staff_id`
  - `name`
  - `phone`
  - `role_names`
  - `status`
  - `created_at`
  - `last_login_at`
  - `last_login_ip`
- `page`
- `page_size`
- `total`

#### `detail`

请求：

- `staff_id`

#### `create`

请求：

- `name`
- `phone`
- `password`
- `contact_qr`
- `role_ids`

#### `update`

请求：

- `staff_id`
- `name`
- `contact_qr`
- `role_ids`
- `status`

#### `disable`

请求：

- `staff_id`

### `role`

- `POST /api/v1/role/list`
- `POST /api/v1/role/detail`
- `POST /api/v1/role/create`
- `POST /api/v1/role/update`

#### `list`

请求：

- `keyword`
- `page`
- `page_size`

#### `detail`

请求：

- `role_id`

#### `create/update`

请求：

- `role_id`（仅 `update` 必填）
- `name`
- `permission_codes`

### `provider`

- `POST /api/v1/provider/list`
- `POST /api/v1/provider/detail`
- `POST /api/v1/provider/create`
- `POST /api/v1/provider/update`
- `POST /api/v1/provider/disable`

#### `list`

请求：

- `city`
- `name`
- `phone`
- `status`
- `page`
- `page_size`

#### `detail`

请求：

- `provider_id`

#### `create`

请求：

- `city_code`
- `name`
- `phone`
- `doorplate_images`
- `appearance_images`
- `credential_images`

#### `update`

请求：

- `provider_id`
- `city_code`
- `name`
- `phone`
- `doorplate_images`
- `appearance_images`
- `credential_images`
- `status`

#### `disable`

请求：

- `provider_id`

说明：

- `provider_id` 是接口层发房方 ID，正式等于 `hs_lld_landlord._id`。
- 当前阶段 provider 只实现房东账号主体与认证，即 `hs_lld_landlord + hs_lld_auth`。
- `name`、图片、资质、品牌等资料依赖 `hs_lld_profile`，当前阶段不进入实现，不得塞进 `hs_lld_landlord` 或 `hs_usr_user`。

### `house`

- `POST /api/v1/house/list`
- `POST /api/v1/house/detail`
- `POST /api/v1/house/export`

#### `list`

请求：

- `source`
- `provider_id`
- `project_id`
- `asset_mode`
- `city`
- `district`
- `audit_status`
- `online_status`
- `page`
- `page_size`

#### `detail`

请求：

- `listing_id`

#### `export`

第一阶段先保留接口名，具体同步导出还是异步导出，进入实现前在 phase 内单独拍板。

说明：

- 第一阶段 `house` 只做运营视角查看，不做后台直接改服务信息或联系人。

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
