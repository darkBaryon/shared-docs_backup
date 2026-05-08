# 账号与身份（V4）

## 1. 范围

## 2. 状态约定

- `0` 未指定
- `1` 有效
- `-1` 删除/禁用

## 3. 公共字段

见 [common-fields.md](./common-fields.md)。

## 4. 集合定义

### 4.1 `hs_usr_user`

用途：C 端用户主档。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `nickname` | string | 否 | `""` | 用户昵称 |
| `avatar` | string | 否 | `""` | 用户头像 URL |
| `phone` | string | 是 | 无 | 微信授权绑定手机号，用户唯一联系手机号 |
| `city` | string | 否 | `""` | 当前城市 |
| `source_channel` | string | 否 | `"miniapp"` | 注册来源渠道，当前固定小程序 |
| `last_active_at` | int64 | 否 | `0` | 最近活跃时间 |

索引：
- `phone_1`（唯一）
- `status_1_updated_at_-1`

### 4.2 `hs_usr_auth`

用途：微信身份认证绑定记录（MVP）。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `user_id` | objectId | 是 | 无 | 关联 `hs_usr_user._id` |
| `auth_provider` | string | 是 | `"wechat"` | 认证来源，固定微信 |
| `openid` | string | 是 | 无 | 微信用户在当前小程序唯一标识 |
| `unionid` | string | 否 | `""` | 微信开放平台统一标识（有则存） |
| `last_login_at` | int64 | 否 | `0` | 最近登录时间 |
| `last_login_ip` | string | 否 | `""` | 最近登录 IP |

索引：
- `auth_provider_1_openid_1`（唯一）
- `auth_provider_1_unionid_1`（唯一，稀疏）
- `user_id_1_status_1`

### 4.3 `hs_usr_profile_ext`

用途：用户找房偏好扩展信息（MVP 当前快照）。

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

待定（非 MVP）：
- 偏好来源追踪（用户确认/AI 抽取/员工修改）。
- 偏好历史版本与置信度机制。

### 4.4 `hs_adm_staff`

用途：后台员工档案（客服/审核/运营/管理员）。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `name` | string | 是 | 无 | 员工姓名 |
| `phone` | string | 是 | 无 | 员工手机号 |
| `email` | string | 否 | `""` | 员工邮箱 |
| `contact_qr_code` | string | 否 | `""` | 含义待确认（当前需求描述不足） |
| `last_login_at` | int64 | 否 | `0` | 最近登录时间 |
| `last_login_ip` | string | 否 | `""` | 最近登录 IP |
| `created_by` | objectId | 否 | 无 | 创建人（员工） |

索引：
- `phone_1`
- `status_1_updated_at_-1`

### 4.5 `hs_adm_role`

用途：角色定义。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `role_name` | string | 是 | 无 | 角色名称 |
| `role_code` | string | 是 | 无 | 角色编码（系统唯一） |
| `description` | string | 否 | `""` | 角色描述 |
| `is_system` | int | 是 | `0` | 是否系统内置角色（1是0否） |

索引：
- `role_code_1`（唯一）
- `status_1_updated_at_-1`

### 4.6 `hs_adm_permission`

用途：权限点定义。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `permission_name` | string | 是 | 无 | 权限名称 |
| `permission_code` | string | 是 | 无 | 权限编码（系统唯一） |
| `module` | string | 是 | 无 | 所属模块（house/audit/lead/ops/user） |
| `action` | string | 否 | `""` | 权限动作（view/create/update/delete/review） |
| `description` | string | 否 | `""` | 权限描述 |

索引：
- `permission_code_1`（唯一）
- `module_1_status_1`

### 4.7 `hs_adm_staff_role`

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

### 4.8 `hs_adm_role_permission`

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

### 4.9 `hs_adm_login_log`

用途：后台登录审计日志。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `staff_id` | objectId | 是 | 无 | 登录员工 |
| `login_at` | int64 | 是 | 当前时间秒 | 登录时间 |
| `login_ip` | string | 是 | 无 | 登录 IP |
| `user_agent` | string | 否 | `""` | 客户端标识 |
| `login_result` | int | 是 | 无 | 登录结果，`1`成功、`-1`失败 |
| `remark` | string | 否 | `""` | 失败原因或备注 |

索引：
- `staff_id_1_login_at_-1`
- `login_at_-1`

## 5. Redis（本模块）

| 键 | 值 | TTL | 备注 |
| --- | --- | --- | --- |
| `hs:sess:{token}` | `user_id` | 会话过期时间 | 用户登录会话 |
| `hs:auth:wechat:{openid}` | `user_id` | 短期（可选） | 微信身份短缓存，加速登录 |

说明：Redis 仅存会话与临时状态，Mongo 存事实数据。

## 6. 角色与权限字典（MVP）

### 6.1 角色字典（`hs_adm_role`）

| role_code | role_name | 备注 |
| --- | --- | --- |
| `super_admin` | 超级管理员 | 全部权限 |
| `admin` | 普通管理员 | 业务管理权限（按分配权限集生效） |
| `tester` | 测试账号 | 测试用途，默认只读或受限操作 |

说明：
- `super_admin` 为系统内置角色（`is_system=1`）。
- `admin`、`tester` 为 MVP 基础角色，后续按业务需要再细分。

### 6.2 权限字典（`hs_adm_permission`）

| permission_code | permission_name | module | action | 备注 |
| --- | --- | --- | --- | --- |
| `house.manage` | 房源管理 | house | manage | 房源管理与审核相关操作 |
| `lead.manage` | 客户与预约管理 | lead | manage | 客户消息、预约看房、跟进处理 |
| `marketing.manage` | 营销管理 | marketing | manage | 广告、弹窗、推荐位、推送任务 |
| `publisher.manage` | 发房方管理 | publisher | manage | 发房方资料与资质维护 |
| `user.manage` | 用户与权限管理 | user | manage | 员工管理、角色管理、权限分配 |
| `system.read` | 系统只读 | system | read | 只读访问（测试账号默认） |

说明：
- `hs_adm_role_permission` 通过 `role_id + permission_id` 关联角色与权限。
- 角色至少需要分配 1 个权限。
- 推荐默认分配：
  - `super_admin`：全部权限
  - `admin`：除 `user.manage` 外按需分配
  - `tester`：仅 `system.read`
