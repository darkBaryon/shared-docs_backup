# 发房方管理模块需求文档

## 1. 背景

发房方管理模块属于 admin Phase 2，服务对象是平台运营员工。

第一阶段 provider 只管理发房方账号主体和登录认证，对应：

- `hs_lld_landlord`
- `hs_lld_auth`

`hs_lld_profile` 已有 schema 位置，但当前阶段不进入实现。因此发房方名称、城市、门牌照、外观照、资质图片、品牌资料等业务资料不在本阶段落库，也不能临时塞进 `hs_lld_landlord`、`hs_usr_user` 或房源主数据。

## 2. 目标

第一阶段目标：

1. 有权限员工可以查看发房方账号列表。
2. 有权限员工可以查看发房方账号详情。
3. 有权限员工可以创建可登录 publish 端的发房方账号。
4. 有权限员工可以修改发房方登录手机号。
5. 有权限员工可以禁用发房方账号。
6. 禁用或修改手机号后，发房方旧 publish token 失效。

## 3. 非目标

第一阶段不做：

- 发房方名称维护。
- 城市、门牌照、外观照、资质图片维护。
- 品牌名称、品牌 logo、品牌封面和品牌介绍维护。
- 发房方业务资料审核。
- 发房方角色权限体系。
- 发房方密码重置短信验证码流程。
- 删除发房方账号。

## 4. 用户故事

### 4.1 查看发房方账号列表

作为运营员工，我需要查看平台创建过的发房方账号，以便确认账号状态和登录情况。

验收：

- 可以按手机号筛选。
- 可以按状态筛选。
- 可以分页查看。
- 列表展示 `provider_id`、手机号、状态、创建人、更新人、最近登录时间、最近登录 IP。
- 无 `provider.view` 权限时不能查看。

### 4.2 查看发房方账号详情

作为运营员工，我需要查看单个发房方账号详情，以便确认账号是否可用。

验收：

- 可以看到账号主体信息。
- 可以看到密码更新时间。
- 可以看到最近登录时间和 IP。
- 发房方不存在或已删除时返回明确中文提示。

### 4.3 创建发房方账号

作为运营员工，我需要创建发房方账号，以便发房方可以登录 publish 端录入房源。

验收：

- 手机号必填且唯一。
- 初始密码必填。
- 密码以 bcrypt hash 写入 `hs_lld_auth`。
- 创建成功后返回账号摘要。
- 重复手机号不能创建。
- 无 `provider.edit` 权限时不能创建。

### 4.4 编辑发房方账号

作为运营员工，我需要修改发房方手机号，以便修正账号录入错误。

验收：

- `provider_id` 必填。
- `phone` 传入时必须是合法手机号。
- 新手机号不能和已有发房方账号重复。
- 修改手机号后，发房方旧 publish token 失效。
- 无 `provider.edit` 权限时不能编辑。

### 4.5 禁用发房方账号

作为运营员工，我需要禁用发房方账号，以便阻止该账号继续登录和维护房源。

验收：

- `provider_id` 必填。
- 禁用采用软状态，不物理删除。
- 重复禁用按成功处理。
- 禁用后不能再次登录 publish。
- 禁用后旧 publish token 失效。
- 无 `provider.edit` 权限时不能禁用。

## 5. 业务规则

### 5.1 主键和命名

- 接口层统一使用 `provider_id`。
- `provider_id` 等于 `hs_lld_landlord._id`。
- 后端内部仍使用 landlord 模型和 repository。

### 5.2 数据边界

- `hs_lld_landlord` 只保存账号主体字段。
- `hs_lld_auth` 只保存认证字段。
- 当前阶段不写 `hs_lld_profile`。
- 当前阶段不返回发房方名称、城市、图片、品牌字段。

### 5.3 Session 失效

- publish 登录态中 principal 是 `principal_type=landlord`、`terminal=publish`。
- 修改手机号或禁用账号后，后端必须失效该发房方旧 publish token。
- 发房方重新登录后才能获得新 token。

### 5.4 非事务写法

- 当前开发阶段不启用 Mongo 多集合事务。
- 创建发房方采用非事务顺序写：
  - 先写 `hs_lld_landlord`
  - 再写 `hs_lld_auth`
- 如果认证信息写入失败，后端清理本次创建的发房方主体和认证信息。

## 6. 页面范围

第一阶段页面：

- 发房方账号列表页。
- 发房方账号详情页或抽屉。
- 新建发房方账号弹窗或抽屉。
- 编辑发房方手机号弹窗或抽屉。
- 禁用确认弹窗。

页面字段：

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

## 7. 接口范围

第一阶段接口：

- `POST /api/v1/provider/list`
- `POST /api/v1/provider/detail`
- `POST /api/v1/provider/create`
- `POST /api/v1/provider/update`
- `POST /api/v1/provider/disable`

## 8. 任务拆分

文档：

- 补 provider 需求文档。
- 补 provider 一接口一文档。
- 更新 admin API 和实施计划。

后端：

- 补 landlord repository 的后台查询、更新和回滚能力。
- 补 landlord_auth repository 的批量摘要和回滚能力。
- 新增 `service/admin/provider`。
- 新增 `handler/v1/admin/provider`。
- 接入 wire 和 admin protected routes。

测试：

- service 单测覆盖创建、重复手机号、创建回滚、更新手机号、禁用和 session 失效。
- handler 单测覆盖成功响应、权限拒绝、JSON 错误和 service 错误透传。

## 9. 待后续

- `hs_lld_profile` 进入实现后，再补发房方资料维护。
- 发房方资料字段进入实现后，再恢复城市、名称、图片、品牌等筛选和展示能力。
