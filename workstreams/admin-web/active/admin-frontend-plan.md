# 管理后台前端详细开发计划

本文档只解决三件事：

1. 第一阶段有哪些页面
2. 每个页面落到哪些前端文件
3. 每个页面依赖哪些接口

前置说明：

- admin 前端是新工程。
- 现有同事写的管理前端分支只作为框架结构参考，不继承任何业务逻辑。
- schema 未拍板前，不进入业务页面实现。

## 1. 前端目标目录

管理后台前端建议按下列目录落地：

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
  router/
    index.ts
  stores/
    adminAuth.ts
    permission.ts
  layout/
    AdminLayout.vue
    AdminSidebar.vue
    AdminHeader.vue
  views/
    admin/
      login/
        LoginPage.vue
      staff/
        StaffListPage.vue
        StaffDrawer.vue
      role/
        RoleListPage.vue
        RoleDrawer.vue
      provider/
        ProviderListPage.vue
        ProviderDrawer.vue
        ProviderDetailPage.vue
      house/
        HouseListPage.vue
        HouseDetailDrawer.vue
      launch-audit/
        LaunchAuditListPage.vue
        LaunchAuditDetailPage.vue
        LaunchAuditDecisionDialog.vue
```

说明：

- 列表页和编辑 drawer 分开文件，不把整页全塞进一个大 SFC。
- `api/` 负责请求函数。
- `model/admin/*` 负责 DTO 到页面 ViewModel 转换。
- `stores/` 只放跨页面状态，例如登录态、权限、菜单。

## 2. 路由骨架

第一阶段路由建议固定为：

```text
/login
/forgot-password

/staff
/roles
/providers
/houses
/launch-audits
```

说明：

- 不做花哨首页。
- 登录后默认进入 `/launch-audits` 或 `/houses`，由产品最终选一个。
- 左侧导航只挂第一阶段模块。

## 3. Phase 1：登录 / 员工 / 角色

### 3.1 登录

文件：

```text
src/views/admin/login/LoginPage.vue
src/api/adminAuth.ts
src/stores/adminAuth.ts
src/stores/permission.ts
```

页面职责：

- 登录页：
  - 手机号
  - 密码
  - 登录

调用接口：

- `admin_auth/login`
- `admin_auth/session`
- `admin_auth/logout`

说明：

- `reset_password` 第一阶段后移，因此忘记密码页不进入首批开发。

### 3.2 员工管理

文件：

```text
src/views/admin/staff/StaffListPage.vue
src/views/admin/staff/StaffDrawer.vue
src/api/staff.ts
src/model/admin/staff.ts
```

页面职责：

- 列表筛选
- 新建员工
- 编辑员工
- 停用员工

列表列：

- 员工姓名
- 手机号
- 角色
- 创建时间
- 登录时间
- 登录 IP
- 状态
- 操作

### 3.3 角色权限

文件：

```text
src/views/admin/role/RoleListPage.vue
src/views/admin/role/RoleDrawer.vue
src/api/role.ts
src/model/admin/role.ts
```

页面职责：

- 角色列表
- 新建角色
- 编辑角色权限

关键交互：

- 权限树 checkbox
- 至少选 1 个权限

## 4. Phase 2：发房方管理

文件：

```text
src/views/admin/provider/ProviderListPage.vue
src/views/admin/provider/ProviderDrawer.vue
src/views/admin/provider/ProviderDetailPage.vue
src/api/provider.ts
src/model/admin/provider.ts
```

页面职责：

- 发房方列表
- 创建 / 编辑
- 查看资质资料
- 禁用账号

列表列：

- 城市
- 发房方名称
- 手机号
- 状态
- 房源数量
- 创建时间

详情页分区：

1. 基础信息
2. 资质资料
3. 品牌信息
4. 房源概况

## 5. Phase 3：房源管理

文件：

```text
src/views/admin/house/HouseListPage.vue
src/views/admin/house/HouseDetailDrawer.vue
src/api/house.ts
src/model/admin/house.ts
```

页面职责：

- 平台视角全量房源列表
- 查看房源详情
- 查看审核状态
- 导出

列表筛选：

- 来源系统
- 发房方
- 项目
- 类型
- 审核状态
- 上下架状态

页面形态：

- 主列表页
- 详情 drawer

说明：

- 第一阶段 `house` 只读，不做后台改服务信息、改联系人。

## 6. Phase 4：房源审核

文件：

```text
src/views/admin/launch-audit/LaunchAuditListPage.vue
src/views/admin/launch-audit/LaunchAuditDetailPage.vue
src/views/admin/launch-audit/LaunchAuditDecisionDialog.vue
src/api/launchAudit.ts
src/model/admin/launchAudit.ts
```

页面职责：

- 查看上架审核列表
- 审核详情
- 通过 / 驳回

审核详情页必须显示：

1. 审核单基础信息
2. 当前生效内容
3. 待审核内容
4. 差异摘要
5. 审核备注输入区

## 7. 布局与权限

文件：

```text
src/layout/AdminLayout.vue
src/layout/AdminSidebar.vue
src/layout/AdminHeader.vue
src/stores/permission.ts
```

职责：

- 顶部员工信息 / 退出
- 左侧菜单
- 根据权限隐藏菜单和操作按钮

菜单第一阶段固定为：

- 房源上架审核
- 房源管理
- 发房方管理
- 员工管理
- 角色权限

## 8. 每阶段前端验收

### Phase 1

- 能登录
- 能恢复 session
- 员工列表 / 新增 / 编辑 / 停用可用
- 角色权限编辑可用

### Phase 2

- 发房方列表 / 新增 / 编辑 / 禁用可用
- 发房方详情资料可查看

### Phase 3

- 平台房源列表可筛选
- 房源详情可看
- 审核状态可看

### Phase 4

- 上架审核列表可用
- 审核详情可读
- 通过 / 驳回动作可用

## 9. 开发规则

1. 新页面必须先写进本文档，再落代码。
2. 页面不直接写请求 URL，统一走 `src/api/*`。
3. 页面展示字段统一走 `src/model/admin/*` 做映射。
4. 不复制 publish 页面作为 admin 页面起点；允许复用基础组件，但不复制业务页面语义。
