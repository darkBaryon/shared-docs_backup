# 账号与身份模块

## 1. 模块目标

本模块用于管理系统中的两类身份主体：

- C 端用户：小程序用户、租客、房东
- B 端员工：后台管理员、客服、运营人员、审核人员

该模块负责“谁在系统中、凭什么登录、拥有什么权限”，不负责房源、聊天、客户跟进等业务数据。

## 2. 集合清单

- `usr_user`
- `usr_auth`
- `usr_profile_ext`
- `adm_staff`
- `adm_role`
- `adm_permission`
- `adm_staff_role`
- `adm_login_log`

## 3. 集合设计

### 3.1 `usr_user`

**功能说明**  
小程序用户主档案。用于存放用户在 C 端视角下的基础资料与业务身份信息。

**核心字段**

- `user_id`：用户唯一业务标识
- `nickname`：用户昵称
- `avatar`：头像地址
- `phone`：手机号
- `roles`：用户业务角色，例如租客、房东
- `status`：用户状态，例如正常、禁用
- `city`：当前服务城市
- `created_at`：创建时间
- `updated_at`：更新时间

### 3.2 `usr_auth`

**功能说明**  
用户认证信息。用于存放微信身份、登录方式、绑定关系等认证相关数据。

**核心字段**

- `user_id`：关联 `usr_user.user_id`
- `auth_provider`：认证来源，例如 `wechat`
- `openid`：微信 OpenID
- `unionid`：微信 UnionID
- `phone_verified`：手机号是否已完成验证
- `last_login_at`：最近登录时间
- `last_login_ip`：最近登录 IP
- `created_at`：创建时间
- `updated_at`：更新时间

### 3.3 `usr_profile_ext`

**功能说明**  
用户扩展档案。用于承载偏好、找房画像、补充资料等非认证类信息。

**核心字段**

- `user_id`：关联 `usr_user.user_id`
- `budget_min`：预算下限
- `budget_max`：预算上限
- `preferred_areas`：偏好区域
- `preferred_rent_mode`：偏好租住方式
- `move_in_plan`：计划入住时间
- `remark`：补充备注
- `updated_at`：更新时间

### 3.4 `adm_staff`

**功能说明**  
后台员工档案。用于存放客服、审核、运营、管理员等 B 端人员资料。

**核心字段**

- `staff_id`：员工唯一标识
- `name`：员工姓名
- `phone`：手机号
- `email`：邮箱
- `status`：员工状态
- `department`：所属部门
- `job_title`：岗位名称
- `contact_qr_code`：对外联系二维码
- `created_at`：创建时间
- `updated_at`：更新时间

### 3.5 `adm_role`

**功能说明**  
后台角色定义。用于表示员工在后台系统中的角色集合。

**核心字段**

- `role_id`：角色唯一标识
- `role_name`：角色名称
- `role_code`：角色编码
- `description`：角色说明
- `status`：角色状态
- `created_at`：创建时间
- `updated_at`：更新时间

### 3.6 `adm_permission`

**功能说明**  
后台权限点定义。用于管理后台菜单、操作按钮、审核能力等权限点。

**核心字段**

- `permission_id`：权限唯一标识
- `permission_code`：权限编码
- `permission_name`：权限名称
- `module`：所属模块
- `description`：权限说明
- `created_at`：创建时间

### 3.7 `adm_staff_role`

**功能说明**  
员工与角色关系映射。用于描述一个员工拥有哪些角色。

**核心字段**

- `staff_id`：关联 `adm_staff.staff_id`
- `role_id`：关联 `adm_role.role_id`
- `assigned_by`：分配人
- `assigned_at`：分配时间

### 3.8 `adm_login_log`

**功能说明**  
后台登录日志。用于记录员工登录时间、IP、设备等安全审计信息。

**核心字段**

- `staff_id`：关联 `adm_staff.staff_id`
- `login_at`：登录时间
- `login_ip`：登录 IP
- `user_agent`：客户端标识
- `login_result`：登录结果
- `remark`：补充说明
