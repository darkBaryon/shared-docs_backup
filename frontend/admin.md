# Frontend Admin

后台管理前端是独立运营控制台。本仓库按全新的 admin 前端工程推进，不承载 publish 前端，也不为 publish 前端做兼容设计。

## 页面形态

- 以高密度列表页为核心
- 强筛选、强批量操作、强详情抽屉
- 适合审核、运营、配置类重复工作

## 第一阶段页面

- 登录
- 房源管理
- 房源上架审核
- 发房方管理
- 员工管理
- 角色权限

## 当前实现前提

- admin 前端是新工程。
- publish 前端已迁移到其它仓库，本仓库只实现 admin 前端。
- 从零组织 admin 页面、路由、store、API 和布局，不继承 publish 前端结构或业务逻辑。
- 已可优先进入 `admin_auth`、`staff`、`role`、`provider`、`house` 前端实现。
- `launch_audit` 后端实现和审核权限点未完成前，只做页面设计，不挂可用菜单。

## 详细设计入口

- 前端需求：[../workstreams/admin-web/active/frontend-requirements.md](../workstreams/admin-web/active/frontend-requirements.md)
- 页面设计：[../workstreams/admin-web/active/frontend-page-spec.md](../workstreams/admin-web/active/frontend-page-spec.md)
- 实施计划：[../workstreams/admin-web/active/admin-frontend-plan.md](../workstreams/admin-web/active/admin-frontend-plan.md)

## 目标文件结构

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

## 约定

- 不直接复用发房系统 API 作为后台管理 API。
- 审核、运营、数据维护类能力先对齐 `product/admin.md` 和 `api/admin.md`。
- 页面与模块边界优先按后台业务动作组织，而不是按底层数据库集合组织。
- 只实现 admin 登录态，不保留 publish token、store、layout、router 兼容层。
- 发房方第一阶段只展示账号字段，不展示名称、城市、资质、品牌等 `hs_lld_profile` 字段。
