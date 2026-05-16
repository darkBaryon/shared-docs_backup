# Frontend Admin

后台管理前端是独立运营控制台，不沿用 `publish` 的录入工作台交互。

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
- 可以参考已有管理前端框架仓的结构，但不继承任何业务逻辑。
- schema 未拍板前，不进入业务页面实现。

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
    admin/
      auth.ts
      staff.ts
      role.ts
      provider.ts
      house.ts
      launchAudit.ts
  stores/
    adminAuth.ts
    permission.ts
  layout/
    AdminLayout.vue
    AdminSidebar.vue
    AdminHeader.vue
  views/
    admin/login/LoginPage.vue
    admin/login/ForgotPasswordPage.vue
    admin/staff/StaffListPage.vue
    admin/staff/StaffDrawer.vue
    admin/role/RoleListPage.vue
    admin/role/RoleDrawer.vue
    admin/provider/ProviderListPage.vue
    admin/provider/ProviderDrawer.vue
    admin/provider/ProviderDetailPage.vue
    admin/house/HouseListPage.vue
    admin/house/HouseDetailDrawer.vue
    admin/launch-audit/LaunchAuditListPage.vue
    admin/launch-audit/LaunchAuditDetailPage.vue
    admin/launch-audit/LaunchAuditDecisionDialog.vue
```

## 约定

- 不直接复用发房系统 API 作为后台管理 API。
- 审核、运营、数据维护类能力先对齐 `product/admin.md` 和 `api/admin.md`。
- 页面与模块边界优先按后台业务动作组织，而不是按底层数据库集合组织。
