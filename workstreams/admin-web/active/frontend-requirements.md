# 管理后台前端需求文档

本文档定义 admin 前端第一阶段的产品化前端需求。页面级结构、字段和交互细节见 [frontend-page-spec.md](./frontend-page-spec.md)。

## 1. 当前口径

admin 前端是平台员工使用的独立运营控制台。本仓库按全新的 admin 前端工程推进，不承载 publish 前端，也不为 publish 前端做共存或兼容设计。

publish 前端已迁移到其它仓库，本仓库不需要保留 publish 页面、路由、store、token 或构建入口。

第一阶段只服务当前确认需求：

- 员工登录和 session 恢复。
- 员工、角色、权限管理。
- 发房方账号管理。
- 平台全量房源只读总览。
- 房源上架审核的页面方案预设计。

当前后端实现状态：

- 已有后端实现和 DTO：`admin_auth`、`staff`、`role`、`provider`、`house`。
- 暂无后端实现：`launch_audit`。
- `launch_audit` 权限点暂未进入权限字典，因此前端只能先完成页面设计和接口封装占位，不能进入可交互业务实现。

## 2. 第一阶段目标

1. 员工可以用手机号和密码登录后台。
2. 前端能恢复 session，并以 session 中的权限控制菜单、路由和按钮。
3. 有权限员工可以完成员工新增、编辑、禁用和角色分配。
4. 有权限员工可以完成角色创建、权限查看和权限编辑。
5. 有权限员工可以完成发房方账号创建、手机号编辑、禁用和详情查看。
6. 有权限员工可以查看和筛选全量房源。
7. 上架审核页面完成信息架构和接口依赖设计，等待后端审核单模型和权限点落地。

## 3. 第一阶段不做

- 不做首页数据大屏或营销式首页。
- 不做 forgot password 页面；`admin_auth/reset_password` 第一阶段后移。
- 不做后台直接编辑房源基础信息、服务信息、联系人信息。
- 不做房源导出真实功能；`house/export` 只保留后续接口名。
- 不做发房方名称、城市、图片、资质、品牌资料维护；这些依赖 `hs_lld_profile`，当前阶段不进入实现。
- 不做复杂数据范围权限，例如按城市、发房方、审核组限制可见数据。
- 不做旧后台、旧前端、旧数据库兼容。

## 4. 用户角色

| 角色 | 前端能力 |
| --- | --- |
| `super_admin` | 可以看到并操作第一阶段全部模块。 |
| `ops_admin` | 以发房方、房源、审核运营为核心；具体可见能力以 session `permission_codes` 为准。 |
| `viewer` | 只读查看；具体可见能力以 session `permission_codes` 为准。 |
| 无角色员工 | 可以登录，但只能看到无权限空状态，不能访问业务模块。 |

前端不得根据 `role_code` 硬编码业务授权，必须以 `permission_codes` 判断菜单、路由和按钮。

## 5. 权限规则

### 5.1 菜单权限

| 菜单 | 最低权限 | 备注 |
| --- | --- | --- |
| 员工管理 | `staff.view` | 创建、编辑、禁用按钮需要 `staff.edit`。 |
| 角色权限 | `role.view` | 创建、编辑按钮需要 `role.edit`。 |
| 发房方管理 | `provider.view` | 创建、编辑、禁用按钮需要 `provider.edit`。 |
| 房源管理 | `house.view` | 第一阶段只读。 |
| 房源上架审核 | 待定 | 需要等 `launch_audit` 权限点进入权限字典。 |

### 5.2 路由权限

- 进入受保护路由前必须先有 admin token。
- 有 token 但内存无 session 时，先调用 `admin_auth/session`。
- session 恢复成功后，根据目标路由 `meta.permission` 判断是否允许进入。
- 无权限进入业务路由时展示 403 页面或模块内无权限状态。
- session 失效时清理 admin token 并跳转 `/login`。

### 5.3 按钮权限

- 编辑类按钮必须按 `*.edit` 控制展示和禁用。
- 前端隐藏按钮只是体验优化，不能替代后端校验。
- 操作失败时直接展示后端返回的中文 `error`。

## 6. 登录态与 Token

admin 登录态使用 opaque token，不使用 JWT。

前端要求：

- 只设计 admin 登录态存储，不保留 publish token 兼容逻辑。
- admin token 使用独立明确的 storage key，例如 `admin_token`。
- 请求 admin API 时统一带 `Authorization: Bearer <admin_token>`。
- `admin_auth/login` 成功后写入 admin token，并保存 `principal`、`staff_profile`、`role_codes`、`permission_codes`。
- `admin_auth/session` 每次应用启动或刷新后调用，用于恢复员工资料和最新权限。
- `admin_auth/logout` 只退出当前 token，成功或 token 已失效后均应清理本地登录态。
- 收到 401 或统一未登录错误码时清理 admin token。

## 7. 信息架构

第一阶段路由固定为：

```text
/login

/launch-audits
/houses
/providers
/staff
/roles

/403
```

说明：

- 不实现 `/forgot-password`。
- 登录后默认进入第一个有权限的菜单，推荐顺序：`/launch-audits`、`/houses`、`/providers`、`/staff`、`/roles`。
- 当 `launch_audit` 未可用时，默认进入 `/houses` 或第一个有权限模块。
- 无任何业务权限时进入无权限空状态页。

## 8. 页面体验要求

后台是高密度运营控制台：

- 列表页优先，筛选区紧凑，不做大卡片首页。
- 列表操作以详情抽屉、编辑抽屉和确认弹窗为主。
- 详情页优先展示业务判断需要的信息，避免展示不可操作的历史字段堆叠。
- 表格列要支持空值展示，统一显示 `-`。
- 时间统一格式化为 `YYYY-MM-DD HH:mm`，时间戳为 `0` 或空时显示 `-`。
- 状态统一用标签显示，颜色只表达状态，不表达权限。
- 删除类能力第一阶段不存在，禁用必须用二次确认。

## 9. 通用状态

每个列表页都必须覆盖：

- 首次加载中。
- 筛选查询中。
- 空列表。
- 接口错误。
- 无权限。
- 操作提交中。
- 操作成功后刷新列表或详情。

每个编辑抽屉都必须覆盖：

- 新建 / 编辑模式。
- 表单校验错误。
- 提交中防重复。
- 后端错误提示。
- 关闭前未保存提示可后移，第一阶段不强制。

## 10. API 接入要求

前端请求统一走：

```text
src/api/*
src/model/*
src/stores/adminAuth.ts
src/stores/permission.ts
```

约定：

- 页面不直接写 URL。
- API 文件只做接口调用和 TypeScript 请求/响应类型引用。
- `model/*` 负责把后端 DTO 转为页面 ViewModel，例如状态文案、时间展示、价格展示、权限分组。
- 后端响应统一为 `{ code, error, data }`。
- 请求参数不提交当前员工 `staff_id` 作为操作人，操作人由后端 session 推导。

## 11. 模块范围

### 11.1 登录

目标：

- 手机号密码登录。
- 恢复 session。
- 退出登录。

验收：

- 正确手机号密码能进入后台。
- 错误手机号或密码展示后端统一中文错误。
- 刷新页面后能恢复员工资料和权限。
- 退出后不能继续访问受保护页面。

### 11.2 员工管理

目标：

- 查看员工列表和详情。
- 创建员工。
- 编辑员工基础资料和角色。
- 禁用员工。

限制：

- 手机号创建后第一阶段不支持修改。
- 密码只在创建时录入，不在编辑页面展示或修改。
- 不允许禁用当前登录账号，错误由后端返回。

验收：

- `staff.view` 可以查看列表和详情。
- `staff.edit` 可以创建、编辑、禁用。
- 无角色员工可创建，但创建后只能登录，不能访问业务功能。
- 禁用后列表状态刷新。

### 11.3 角色权限

目标：

- 查看角色列表和详情。
- 创建角色。
- 编辑角色名称、说明和权限。

限制：

- `role_code` 创建后不可修改。
- 权限点来自后端返回的角色详情和权限字典数据；当前没有独立 permission list 接口，首版可从角色详情返回的 `permissions` 结构组织展示。
- `permission_codes` 创建时至少 1 个；更新时传空数组不允许。

验收：

- `role.view` 可以查看。
- `role.edit` 可以创建和编辑。
- 非法权限码错误展示后端中文提示。
- 权限变更后，相关员工旧 token 由后端失效；前端收到 session 失效后重新登录。

### 11.4 发房方管理

目标：

- 查看发房方账号列表和详情。
- 创建发房方账号。
- 修改发房方手机号。
- 禁用发房方账号。

限制：

- 当前阶段只展示账号信息，不展示发房方名称、城市、图片、资质、品牌。
- `provider_id` 等于 `hs_lld_landlord._id`。

验收：

- `provider.view` 可以查看。
- `provider.edit` 可以创建、编辑手机号、禁用。
- 重复手机号、手机号格式错误、禁用失败等直接展示后端中文错误。

### 11.5 房源管理

目标：

- 平台视角查看全量房源列表。
- 按发房方、资产类型、城市、区域、房态、发布状态、审核状态筛选。
- 查看房源详情。

限制：

- 第一阶段只读。
- 不实现编辑、上下架、导出。
- 详情以 `hs_hpd_admin_listing` read model 返回字段为准。

验收：

- `house.view` 可以访问列表和详情。
- 筛选不改变房源状态。
- 无权限时不能进入列表或详情。

### 11.6 房源上架审核

目标：

- 先完成页面设计和文件规划。
- 等审核单模型、接口详情、权限点确认后进入实现。

限制：

- 暂不接真实 API。
- 暂不展示入口给无明确审核权限的账号。
- 不把审核动作临时映射为 `house.edit` 或其它已有权限。

验收：

- 后续 `launch_audit` 接口和权限落地后，可以按页面文档直接实现。

## 12. 开发阶段

### Phase A：admin 基础框架

- 独立 admin token。
- admin 登录页。
- admin session store。
- admin layout、菜单和权限守卫。
- 403 / 无权限空状态。

### Phase B：组织体系

- staff API、model、列表、详情抽屉、创建/编辑抽屉。
- role API、model、列表、详情/编辑抽屉。

### Phase C：发房方账号

- provider API、model、列表、详情抽屉、创建/编辑抽屉、禁用确认。

### Phase D：房源总览

- house API、model、列表、详情抽屉。
- 状态字典和筛选项。

### Phase E：上架审核

- 等 `launch_audit` 后端实现和权限字典补齐后开发。

## 13. 文件规划

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
  stores/
    adminAuth.ts
    permission.ts
  utils/auth/
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

- 本仓库按 admin-only 结构组织目录，不保留 publish 页面、布局、store、token 工具或路由兼容层。
- 可以复用 Element Plus 基础组件和通用格式化工具。
- 不复制 publish 的业务页面结构作为 admin 页面起点。

## 14. 待确认问题

1. `launch_audit` 权限点名称：建议后端补 `launch_audit.view`、`launch_audit.review` 或同等明确权限。
2. 角色权限页面是否需要独立 `permission/list` 接口；当前 role detail 返回权限详情，但创建角色时缺少完整可选权限源。
3. 房源状态枚举的前端中文文案是否以 schema 状态字典为准；进入实现前需沉淀到 `model/house.ts`。
