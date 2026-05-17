# 角色权限模块需求文档

## 1. 背景

角色权限模块属于 admin Phase 1 组织体系，服务对象是平台员工。

该模块解决两件事：

- 平台可以定义后台角色。
- 平台可以给角色分配权限，员工通过角色获得后台功能权限。

角色权限模块不直接处理员工登录、员工资料、发房方、房源或审核业务，只提供后台权限组织能力。

## 2. 目标

第一阶段目标：

1. 有权限员工可以查看角色列表。
2. 有权限员工可以查看角色详情。
3. 有权限员工可以创建角色并分配权限。
4. 有权限员工可以编辑角色名称、说明和权限。
5. 角色权限变更后，已分配该角色员工的旧 admin token 失效，避免旧权限继续生效。

## 3. 非目标

第一阶段不做：

- 角色删除。
- 权限点在线创建、编辑或删除。
- 复杂权限树配置后台。
- 数据范围权限，例如只看某城市、某发房方、某审核组。
- 系统角色保护的完整治理能力，例如禁止编辑 `super_admin`。
- 审核模块权限点细化；`launch_audit` 实现前单独补齐。

## 4. 用户故事

### 4.1 查看角色

作为后台管理员，我需要查看当前有哪些角色，以便理解员工权限组织方式。

验收：

- 可以看到角色名称、角色编码、权限摘要、创建时间和更新时间。
- 可以按角色名称或角色编码关键词搜索。
- 无 `role.view` 权限时不能查看。

### 4.2 查看角色详情

作为后台管理员，我需要查看某个角色拥有的完整权限，以便确认权限配置是否正确。

验收：

- 可以看到角色基础信息。
- 可以看到完整权限码列表。
- 可以看到权限名称、模块和动作。
- 角色不存在或停用时返回明确中文提示。

### 4.3 创建角色

作为后台管理员，我需要创建新角色并分配权限，以便给员工配置不同操作范围。

验收：

- `role_name` 必填。
- `role_code` 必填且唯一。
- `permission_codes` 必填且至少 1 个。
- 非法或停用权限码不能保存。
- 创建成功后返回角色详情。
- 无 `role.edit` 权限时不能创建。

### 4.4 编辑角色

作为后台管理员，我需要调整角色名称、说明和权限，以便后台权限随业务变化更新。

验收：

- 可以编辑角色名称。
- 可以编辑角色说明。
- 可以替换角色权限。
- `role_code` 创建后不可修改。
- 权限列表不传表示不调整权限。
- 权限列表传空数组不允许。
- 权限变更后，使用该角色的员工旧 admin token 失效。
- 无 `role.edit` 权限时不能编辑。

## 5. 业务规则

### 5.1 角色编码

- `role_code` 是稳定业务编码。
- `role_code` 全局唯一。
- `role_code` 创建后不可修改。
- `role_code` 使用小写字母、数字和下划线。

### 5.2 权限来源

- 权限点来自 `hs_adm_permission`。
- 后台 role 接口只接受已存在且有效的 `permission_code`。
- 前端不能提交任意字符串让后端直接生成权限。

### 5.3 Session 失效

- admin 登录态中包含权限快照。
- 角色权限变更后，后端必须失效已分配该角色员工的旧 admin token。
- 员工重新登录或重新建立 session 后才能拿到新权限。

### 5.4 非事务写法

- 当前开发阶段不启用 Mongo 多集合事务。
- 创建角色采用非事务顺序写：先写 `hs_adm_role`，再写 `hs_adm_role_permission`。
- 如果权限关系写入失败，后端清理本次创建的角色和权限关系。
- 不为开发环境引入事务抽象。

## 6. 页面范围

第一阶段页面：

- 角色列表页。
- 角色详情抽屉或详情页。
- 新建角色弹窗或抽屉。
- 编辑角色权限弹窗或抽屉。

页面字段：

- `role_name`
- `role_code`
- `description`
- `permission_codes`
- `permission_names`
- `created_at`
- `updated_at`

## 7. 接口范围

第一阶段接口：

- `POST /api/v1/role/list`
- `POST /api/v1/role/detail`
- `POST /api/v1/role/create`
- `POST /api/v1/role/update`

接口契约：

- [role/list](../../../backend/interfaces/admin/role_list.md)
- [role/detail](../../../backend/interfaces/admin/role_detail.md)
- [role/create](../../../backend/interfaces/admin/role_create.md)
- [role/update](../../../backend/interfaces/admin/role_update.md)

## 8. 任务拆分

文档：

- 补产品需求。
- 补接口文档。
- 补后端实施计划。

后端：

- 补 `repository/adm` role 查询和写入能力。
- 补 `repository/adm` permission 按 code 查询能力。
- 补 `repository/adm` role_permission 创建、替换和回滚能力。
- 补 `service/admin/role`。
- 补 `handler/v1/admin/role`。
- 接入 wire 和 admin protected routes。

测试：

- service 单测覆盖创建、非法权限、创建回滚、权限替换和 session 失效。
- handler 单测覆盖成功响应、权限拒绝、JSON 错误和 service 错误透传。
- 暂不新增真实 Mongo / Redis 集成测试，后续按 admin 模块统一补。

## 9. 当前实现状态

已实现：

- `role/list`
- `role/detail`
- `role/create`
- `role/update`
- 基础 handler / service 单测
- 权限变更后的 admin token 失效

待后续：

- 更完整的系统角色保护规则。
- 权限树展示结构。
- 角色删除或停用能力，如确需使用需单独设计。
