# 账号与身份（V4）

## 1. 范围

本模块只定义三类身份主体及其认证、资料、权限关系：

- `usr`：小程序 C 端用户
- `lld`：发房端房东
- `adm`：管理后台员工

约束：

- `miniapp user != publish landlord != admin staff`
- 主体资料与认证信息必须分表，不把密码、登录安全字段塞进主体表
- 只服务当前确认需求，不为历史库和旧端做兼容设计

## 2. 状态约定

- `0` 未指定
- `1` 有效
- `-1` 删除 / 禁用

## 3. 公共字段

见 [common-fields.md](./common-fields.md)。

## 4. 集合定义

### 4.1 `hs_usr_user`

用途：小程序用户主档。只服务 `miniapp`，不承载房东主体。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `nickname` | string | 否 | `""` | 用户昵称 |
| `avatar` | string | 否 | `""` | 用户头像 URL |
| `phone` | string | 是 | 无 | 小程序用户唯一联系手机号 |
| `city` | string | 否 | `""` | 当前城市 |
| `source_channel` | string | 否 | `"miniapp"` | 注册来源，当前固定小程序 |
| `last_active_at` | int64 | 否 | `0` | 最近活跃时间 |

索引：

- `phone_1`（唯一）
- `status_1_updated_at_-1`

### 4.2 `hs_usr_auth`

用途：小程序用户认证表。当前只承载微信登录绑定信息。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `user_id` | objectId | 是 | 无 | 关联 `hs_usr_user._id` |
| `auth_provider` | string | 是 | `"wechat"` | 当前固定微信 |
| `openid` | string | 是 | 无 | 微信用户在当前小程序唯一标识 |
| `unionid` | string | 否 | `""` | 微信开放平台统一标识（有则存） |
| `last_login_at` | int64 | 否 | `0` | 最近登录时间 |
| `last_login_ip` | string | 否 | `""` | 最近登录 IP |

索引：

- `auth_provider_1_openid_1`（唯一）
- `auth_provider_1_unionid_1`（唯一，稀疏）
- `user_id_1_status_1`

### 4.3 `hs_usr_profile_ext`

用途：小程序用户找房偏好扩展信息（MVP 当前快照）。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `user_id` | objectId | 是 | 无 | 关联 `hs_usr_user._id` |
| `budget_min` | int | 否 | `0` | 预算下限 |
| `budget_max` | int | 否 | `0` | 预算上限 |
| `preferred_areas` | array<string> | 否 | `[]` | 偏好区域列表 |
| `preferred_rent_mode` | string | 否 | `""` | 偏好租住方式 |
| `move_in_plan` | string | 否 | `""` | 计划入住时间描述 |
| `remark` | string | 否 | `""` | 用户偏好补充备注 |

索引：

- `user_id_1`（唯一）

### 4.4 `hs_lld_landlord`

用途：发房端房东主体表。只表达“这个房东是谁”，不承载密码等认证字段。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `phone` | string | 是 | 无 | 房东登录账号手机号，主体唯一标识 |
| `created_by_staff_id` | objectId | 否 | 无 | 后台创建人 |
| `updated_by_staff_id` | objectId | 否 | 无 | 最近更新人 |

索引：

- `phone_1`（唯一）
- `status_1_updated_at_-1`

### 4.5 `hs_lld_auth`

用途：房东认证表。认证信息独立存放，不混入 `hs_lld_landlord`。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `landlord_id` | objectId | 是 | 无 | 关联 `hs_lld_landlord._id` |
| `auth_type` | string | 是 | `"password"` | 当前固定账号密码登录 |
| `password_hash` | string | 是 | 无 | 密码哈希值 |
| `password_updated_at` | int64 | 否 | `0` | 最近一次改密时间 |
| `last_login_at` | int64 | 否 | `0` | 最近登录时间 |
| `last_login_ip` | string | 否 | `""` | 最近登录 IP |

索引：

- `landlord_id_1`（唯一）
- `auth_type_1_status_1`

说明：

- `password_hash` 当前固定存 bcrypt hash，不单独存 salt；
- `status=-1` 表示该认证方式不可用，不影响 `hs_lld_landlord` 主体资料本身；
- 登录时先按 `hs_lld_landlord.phone` 定位主体，再读取有效 `hs_lld_auth` 校验密码。

### 4.6 `hs_lld_profile`

用途：房东业务资料表。一对一挂在房东主体上，承载发房方资料与品牌信息。

当前阶段说明：

- 本表属于 `lld` 模块正式 schema 的一部分；
- 当前阶段先只落 `hs_lld_landlord` 与 `hs_lld_auth`；
- `hs_lld_profile` 暂不进入实现范围，先作为后续扩展表保留。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `landlord_id` | objectId | 是 | 无 | 关联 `hs_lld_landlord._id` |
| `landlord_name` | string | 是 | 无 | 房东 / 发房方显示名称 |
| `city_code` | string | 否 | `""` | 所在城市编码 |
| `city_name` | string | 否 | `""` | 所在城市名称 |
| `doorplate_images` | array<string> | 否 | `[]` | 门牌照 |
| `appearance_images` | array<string> | 否 | `[]` | 外观照 |
| `credential_images` | array<string> | 否 | `[]` | 资质 / 凭证资料 |
| `brand_name` | string | 否 | `""` | 品牌名 |
| `brand_logo` | string | 否 | `""` | 品牌 Logo |
| `brand_cover` | string | 否 | `""` | 品牌封面图 |
| `brand_intro` | string | 否 | `""` | 品牌简介 |
| `created_by_staff_id` | objectId | 否 | 无 | 后台创建人 |
| `updated_by_staff_id` | objectId | 否 | 无 | 最近更新人 |

索引：

- `landlord_id_1`（唯一）
- `city_code_1_status_1`
- `landlord_name_1`

### 4.7 `hs_adm_staff`

用途：后台员工主体表。只存员工资料，不承载密码等认证字段。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `name` | string | 是 | 无 | 员工姓名 |
| `phone` | string | 是 | 无 | 员工手机号 |
| `email` | string | 否 | `""` | 员工邮箱 |
| `department` | string | 否 | `""` | 所属部门 |
| `job_title` | string | 否 | `""` | 岗位名称 |
| `contact_qr_code` | string | 否 | `""` | 对外联系二维码 URL |
| `created_by_staff_id` | objectId | 否 | 无 | 创建人（员工） |

索引：

- `phone_1`（唯一）
- `status_1_updated_at_-1`

### 4.8 `hs_adm_staff_auth`

用途：后台员工认证表。后台密码登录信息必须独立存放。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `staff_id` | objectId | 是 | 无 | 关联 `hs_adm_staff._id` |
| `auth_type` | string | 是 | `"password"` | 当前固定账号密码登录 |
| `password_hash` | string | 是 | 无 | 密码哈希值 |
| `password_updated_at` | int64 | 否 | `0` | 最近一次改密时间 |
| `last_login_at` | int64 | 否 | `0` | 最近登录时间 |
| `last_login_ip` | string | 否 | `""` | 最近登录 IP |

索引：

- `staff_id_1`（唯一）
- `auth_type_1_status_1`

说明：

- `password_hash` 当前固定存 bcrypt hash，不单独存 salt；
- `status=-1` 表示该认证方式不可用，不影响 `hs_adm_staff` 主体资料本身；
- 登录时先按 `hs_adm_staff.phone` 定位主体，再读取有效 `hs_adm_staff_auth` 校验密码。

### 4.9 `hs_adm_role`

用途：后台角色定义。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `role_name` | string | 是 | 无 | 角色名称 |
| `role_code` | string | 是 | 无 | 角色编码（系统唯一） |
| `description` | string | 否 | `""` | 角色描述 |
| `is_system` | int | 是 | `0` | 是否系统内置角色（1是0否） |

索引：

- `role_code_1`（唯一）
- `status_1_updated_at_-1`

### 4.10 `hs_adm_permission`

用途：后台权限点定义。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `permission_name` | string | 是 | 无 | 权限名称 |
| `permission_code` | string | 是 | 无 | 权限编码（系统唯一） |
| `module` | string | 是 | 无 | 所属模块（`staff` / `role` / `provider` / `house`） |
| `action` | string | 否 | `""` | 权限动作（`view` / `edit` / `export`） |
| `description` | string | 否 | `""` | 权限描述 |

索引：

- `permission_code_1`（唯一）
- `module_1_status_1`

### 4.11 `hs_adm_staff_role`

用途：员工-角色关系。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `staff_id` | objectId | 是 | 无 | 关联员工 |
| `role_id` | objectId | 是 | 无 | 关联角色 |
| `assigned_by` | objectId | 否 | 无 | 分配人 |
| `assigned_at` | int64 | 是 | 当前时间秒 | 分配时间 |

索引：

- `staff_id_1_role_id_1`（唯一）
- `role_id_1_status_1`

### 4.12 `hs_adm_role_permission`

用途：角色-权限关系。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `role_id` | objectId | 是 | 无 | 关联角色 |
| `permission_id` | objectId | 是 | 无 | 关联权限 |
| `assigned_by` | objectId | 否 | 无 | 分配人 |
| `assigned_at` | int64 | 是 | 当前时间秒 | 分配时间 |

索引：

- `role_id_1_permission_id_1`（唯一）
- `permission_id_1_status_1`

### 4.13 `hs_adm_login_log`

用途：后台登录审计日志。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `staff_id` | objectId | 是 | 无 | 登录员工 |
| `login_at` | int64 | 是 | 当前时间秒 | 登录时间 |
| `login_ip` | string | 是 | 无 | 登录 IP |
| `user_agent` | string | 否 | `""` | 客户端标识 |
| `login_result` | int | 是 | 无 | 登录结果，`1` 成功、`-1` 失败 |
| `remark` | string | 否 | `""` | 失败原因或备注 |

索引：

- `staff_id_1_login_at_-1`
- `login_at_-1`

## 5. Redis（本模块）

| 键 | 值 | TTL | 备注 |
| --- | --- | --- | --- |
| `hs:sess:{token}` | structured principal JSON | 会话过期时间 | 登录会话 |
| `hs:auth:wechat:{openid}` | `user_id` | 短期（可选） | 微信登录短缓存 |

说明：

- Redis 只存会话与临时状态，Mongo 存事实数据。
- `hs:sess:{token}` 对应结构化 session principal，而不是裸 ID。
- 当前三类 principal 约定：
  - `miniapp`：`principal_type=user`
  - `publish`：`principal_type=landlord`
  - `admin`：`principal_type=staff`

## 6. 角色与权限字典（Admin MVP）

### 6.1 角色字典（`hs_adm_role`）

| role_code | role_name | 备注 |
| --- | --- | --- |
| `super_admin` | 超级管理员 | 全部权限 |
| `ops_admin` | 运营管理员 | 发房方与房源运营 |
| `viewer` | 只读账号 | 只读访问 |

说明：

- `super_admin` 为系统内置角色（`is_system=1`）。
- 其它角色按当前管理后台 MVP 范围定义，不预留历史兼容角色。

### 6.2 权限字典（`hs_adm_permission`）

| permission_code | permission_name | module | action | 备注 |
| --- | --- | --- | --- | --- |
| `staff.view` | 查看员工 | staff | view | 员工列表 / 详情查看 |
| `staff.edit` | 编辑员工 | staff | edit | 创建、编辑、禁用员工 |
| `role.view` | 查看角色 | role | view | 角色与权限查看 |
| `role.edit` | 编辑角色 | role | edit | 创建、编辑角色与授权 |
| `provider.view` | 查看发房方 | provider | view | 发房方列表 / 详情查看 |
| `provider.edit` | 编辑发房方 | provider | edit | 创建、编辑、禁用发房方 |
| `house.view` | 查看房源 | house | view | 后台房源运营总览 |
| `house.export` | 导出房源 | house | export | 房源列表导出 |

说明：

- `hs_adm_role_permission` 通过 `role_id + permission_id` 关联角色与权限。
- 当前只定义管理后台 MVP 所需权限点，不预留无需求的扩展权限。
