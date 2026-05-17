# 管理后台前端详细开发计划

本文档负责 admin 前端的实施顺序和文件级计划。

详细需求与页面设计拆分维护：

- [frontend-requirements.md](./frontend-requirements.md)：前端需求、权限、登录态、范围边界。
- [frontend-page-spec.md](./frontend-page-spec.md)：页面路由、字段、接口、交互和验收。

当前口径：

- admin 前端是平台员工独立运营控制台。本仓库按全新的 admin-only 前端工程推进，不沿用 publish 的页面、路由、store 或 token 逻辑。
- publish 前端已迁移到其它仓库，本仓库从零实现 admin 前端。
- 已可对接后端模块：`admin_auth`、`staff`、`role`、`provider`、`house`。
- `launch_audit` 只有 API 骨架，暂无后端实现和权限点，前端先做页面设计，不进入可用业务实现。
- `admin_auth/reset_password`、`house/export`、发房方资料维护不进入第一阶段。

## 1. 前端目标目录

```text
src/
  api/
    adminAuth.ts
    staff.ts
    role.ts
    provider.ts
    house.ts
    launchAudit.ts
  model/
    auth.ts
    staff.ts
    role.ts
    provider.ts
    house.ts
    launchAudit.ts
    permission.ts
  router/
    index.ts
  stores/
    adminAuth.ts
    permission.ts
  utils/
    auth/
      adminAuthToken.ts
  layout/
    AdminLayout.vue
    AdminSidebar.vue
    AdminHeader.vue
  views/
    login/LoginPage.vue
    staff/StaffListPage.vue
    staff/StaffDrawer.vue
    role/RoleListPage.vue
    role/RoleDrawer.vue
    provider/ProviderListPage.vue
    provider/ProviderDrawer.vue
    house/HouseListPage.vue
    house/HouseDetailDrawer.vue
    launch-audit/LaunchAuditListPage.vue
    launch-audit/LaunchAuditDetailPage.vue
    launch-audit/LaunchAuditDecisionDialog.vue
```

说明：

- `api/` 负责后台请求函数。
- `model/*` 负责 DTO 到页面 ViewModel 转换。
- `stores/` 只放跨页面状态，例如登录态、权限、菜单。
- `layout/` 只承载 admin 控制台布局。

## 2. 路由骨架

```text
/login

/staff
/roles
/providers
/houses
/launch-audits
/403
```

说明：

- 不做花哨首页。
- 不做 `/forgot-password`，`admin_auth/reset_password` 第一阶段后移。
- 登录后默认进入第一个有权限模块；`launch_audit` 未实现前优先进入 `/houses`。
- 左侧导航只挂第一阶段模块，并按权限过滤。

## 3. Phase A：admin 基础框架

### 3.1 目标

Phase A 只解决后台可登录、可恢复 session、可按权限进入页面框架的问题，不进入员工、角色、发房方和房源业务页面实现。

### 3.2 任务清单

1. 入口清理：
   - `App.vue` 只负责渲染 `RouterView`。
   - `router/index.ts` 只注册 admin 路由。
   - 不保留 publish 路由、菜单和登录态兼容逻辑。
2. 登录态：
   - 新增 `utils/auth/adminAuthToken.ts`。
   - admin token storage key 使用 `house_manager_admin_token`。
   - session 失效时清理 token 并跳转 `/login`.
3. API 与模型：
   - 新增 `model/auth.ts`。
   - 新增 `api/adminAuth.ts`。
   - 请求统一使用 `Authorization: Bearer <admin_token>`。
4. Store：
   - 新增 `stores/adminAuth.ts`，管理 token、principal、staff profile、roles、permissions。
   - 新增 `stores/permission.ts`，提供 `hasPermission`、`filterMenus` 等权限判断。
5. 路由守卫：
   - `/login` 为公开路由。
   - 其它路由需要登录。
   - 有 token 但无 session 时，先调用 `admin_auth/session`。
   - 路由 meta 带 `permission` 时按权限拦截。
6. 页面框架：
   - 新增 `layout/AdminLayout.vue`、`layout/AdminSidebar.vue`、`layout/AdminHeader.vue`。
   - 新增 `views/login/LoginPage.vue`。
   - 新增 `views/error/ForbiddenPage.vue`。
   - 业务模块先用占位页承接路由。

### 3.3 验收清单

- 无 token 访问受保护路由会跳转 `/login`。
- 登录成功后保存 admin token，并进入第一个有权限模块。
- 刷新页面会调用 `admin_auth/session` 恢复登录态。
- 无权限访问业务路由会进入 `/403`。
- 退出登录会调用 `admin_auth/logout` 并清理本地状态。
- 代码中不再引用 publish auth store、publish token、publish 路由菜单。

文件：

```text
src/views/login/LoginPage.vue
src/api/adminAuth.ts
src/model/auth.ts
src/stores/adminAuth.ts
src/stores/permission.ts
src/utils/auth/adminAuthToken.ts
src/layout/AdminLayout.vue
src/layout/AdminSidebar.vue
src/layout/AdminHeader.vue
```

页面职责：

- 手机号密码登录。
- session 恢复。
- 退出登录。
- 只实现 admin token，不保留 publish token 兼容逻辑。
- 根据 `permission_codes` 控制菜单、路由和按钮。
- 无权限时展示 403 或无权限空状态。

调用接口：

- `POST /api/v1/admin_auth/login`
- `POST /api/v1/admin_auth/session`
- `POST /api/v1/admin_auth/logout`

验收：

- 能登录。
- 能恢复 session。
- 退出后不能访问业务页面。
- 刷新和重新打开页面后只恢复 admin session。

## 4. Phase B：员工与角色

### 4.1 员工管理

文件：

```text
src/views/staff/StaffListPage.vue
src/views/staff/StaffDrawer.vue
src/api/staff.ts
src/model/staff.ts
```

页面职责：

- 员工列表筛选。
- 查看员工详情。
- 新建员工。
- 编辑员工基础资料和角色。
- 禁用员工。

调用接口：

- `POST /api/v1/staff/list`
- `POST /api/v1/staff/detail`
- `POST /api/v1/staff/create`
- `POST /api/v1/staff/update`
- `POST /api/v1/staff/disable`
- `POST /api/v1/role/list`，用于角色下拉。

权限：

- `staff.view`：列表、详情。
- `staff.edit`：新建、编辑、禁用。

说明：

- 手机号创建后第一阶段不支持修改。
- 密码只在创建时录入，不在编辑页展示或修改。

### 4.2 角色权限

文件：

```text
src/views/role/RoleListPage.vue
src/views/role/RoleDrawer.vue
src/api/role.ts
src/model/role.ts
src/model/permission.ts
```

页面职责：

- 角色列表。
- 查看角色详情。
- 新建角色。
- 编辑角色名称、说明和权限。

调用接口：

- `POST /api/v1/role/list`
- `POST /api/v1/role/detail`
- `POST /api/v1/role/create`
- `POST /api/v1/role/update`

权限：

- `role.view`：列表、详情。
- `role.edit`：新建、编辑。

说明：

- `role_code` 创建后不可修改。
- 创建角色时 `permission_codes` 至少 1 个。
- 当前缺少独立 `permission/list` 接口，进入实现前需要确认权限选择数据源。

## 5. Phase C：发房方账号

文件：

```text
src/views/provider/ProviderListPage.vue
src/views/provider/ProviderDrawer.vue
src/api/provider.ts
src/model/provider.ts
```

页面职责：

- 发房方账号列表。
- 查看账号详情。
- 创建账号。
- 编辑手机号。
- 禁用账号。

调用接口：

- `POST /api/v1/provider/list`
- `POST /api/v1/provider/detail`
- `POST /api/v1/provider/create`
- `POST /api/v1/provider/update`
- `POST /api/v1/provider/disable`

权限：

- `provider.view`：列表、详情。
- `provider.edit`：创建、编辑、禁用。

说明：

- 当前阶段只实现 `hs_lld_landlord + hs_lld_auth`。
- 不展示城市、发房方名称、资质资料、品牌资料、房源数量。
- `hs_lld_profile` 进入实现后再补资料维护页面。

## 6. Phase D：房源管理总览

文件：

```text
src/views/house/HouseListPage.vue
src/views/house/HouseDetailDrawer.vue
src/api/house.ts
src/model/house.ts
```

页面职责：

- 平台视角全量房源列表。
- 按发房方、资产类型、城市、区域、房态、发布状态、审核状态筛选。
- 查看房源详情。
- 查看审核状态。

调用接口：

- `POST /api/v1/house/list`
- `POST /api/v1/house/detail`

权限：

- `house.view`：列表、详情。

说明：

- 第一阶段 `house` 只读，不做后台改服务信息、改联系人。
- 不实现导出。

## 7. Phase E：房源上架审核

文件：

```text
src/views/launch-audit/LaunchAuditListPage.vue
src/views/launch-audit/LaunchAuditDetailPage.vue
src/views/launch-audit/LaunchAuditDecisionDialog.vue
src/api/launchAudit.ts
src/model/launchAudit.ts
```

页面职责：

- 查看上架审核列表。
- 查看审核详情。
- 审核通过 / 驳回。

计划接口：

- `POST /api/v1/launch_audit/list`
- `POST /api/v1/launch_audit/detail`
- `POST /api/v1/launch_audit/approve`
- `POST /api/v1/launch_audit/reject`

当前状态：

- 后端暂无实现。
- 审核权限点未进入权限字典。
- 前端只保留页面设计和文件计划，暂不挂可用菜单。

实现前置：

1. 补审核权限点。
2. 补审核单详情响应字段。
3. 补当前生效内容、待审核内容、差异摘要结构。
4. 补审核状态枚举。

## 8. 每阶段前端验收

### Phase A

- 能登录。
- 能恢复 session。
- 能按权限生成菜单。
- 退出登录可用。
- 无权限页面可用。

### Phase B

- 员工列表 / 详情 / 新增 / 编辑 / 禁用可用。
- 角色列表 / 详情 / 新增 / 编辑权限可用。

### Phase C

- 发房方账号列表 / 详情 / 新增 / 编辑手机号 / 禁用可用。
- 不展示当前阶段没有数据支撑的资料字段。

### Phase D

- 平台房源列表可筛选。
- 房源详情可看。
- 审核状态可看。
- 不提供导出和编辑。

### Phase E

- `launch_audit` 后端实现后，上架审核列表可用。
- 审核详情可读。
- 通过 / 驳回动作可用。

## 9. 开发规则

1. 新页面必须先写进本文档或 [frontend-page-spec.md](./frontend-page-spec.md)，再落代码。
2. 页面不直接写请求 URL，统一走 `src/api/*`。
3. 页面展示字段统一走 `src/model/*` 做映射。
4. 不复制 publish 页面作为 admin 页面起点。
5. 不保留 publish 端 token、store、layout、router 的兼容层。
