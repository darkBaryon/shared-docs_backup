# Admin Web Status

## 当前阶段

管理后台处于 Phase 1 后端先行、前端设计收口阶段。`admin_auth`、`staff`、`role`、`provider`、`house` 已有第一批后端实现和 DTO，`launch_audit` 仍需先补审核单模型、权限点和接口详情再进入代码。

当前不能直接全面开工的原因：

- `launch_audit` 还缺接口级需求和验收标准。
- `launch_audit` 审核权限点还未写入权限字典。
- 上架审核单状态流已初步定义，但进入实现前仍需和 publish 提交流程对齐。
- 角色权限页面缺少独立权限字典列表接口，前端实现前需要确认权限选择数据源。

## 当前结论

- 管理后台是独立前端，不是 `publish` 的增强版。
- publish 前端已迁移到其它仓库，当前前端仓库按 admin-only 工程推进，不做 publish 共存或兼容。
- 管理后台服务对象是平台员工，核心职责是审核、运营、分发、跟进、组织与权限管理。
- 管理后台第一阶段产品需求已补充到 `product/admin.md`，后续接口设计必须以该文档为业务边界。
- 管理后台后端独立走：

```text
handler/v1/admin/*
  -> service/admin/*
```

- 可以复用 `domain/*` 和 `repository/*`，但不复用 `service/miniapp/*` 或 `service/publish/*`。
- 发房方接口层 `provider_id` 已确定为 `hs_lld_landlord._id`。
- 当前阶段发房方只实现 `hs_lld_landlord + hs_lld_auth`，`hs_lld_profile` 暂不进入实现。
- 管理后台前端详细需求已拆到 `frontend-requirements.md`，页面设计已拆到 `frontend-page-spec.md`。

## 第一阶段 MVP

第一阶段只做后台最小闭环：

1. 员工登录
2. 员工 / 角色 / 权限管理
3. 发房方管理
4. 房源管理总览
5. 房源上架审核

不在第一阶段实现：

- 客户消息完整工作台
- 预约看房跟进系统
- 营销管理全量功能
- 房源信息变更审核
- 后台直接修改房源服务信息

## 当前入口

- 产品范围：[../../product/admin.md](../../product/admin.md)
- 前端形态：[../../frontend/admin.md](../../frontend/admin.md)
- 接口骨架：[../../api/admin.md](../../api/admin.md)
- 后端边界：[../../backend/admin.md](../../backend/admin.md)
- 总开发计划：[./active/admin-mvp.md](./active/admin-mvp.md)
- 后端详细计划：[./active/admin-backend-plan.md](./active/admin-backend-plan.md)
- 前端详细计划：[./active/admin-frontend-plan.md](./active/admin-frontend-plan.md)
- 前端需求文档：[./active/frontend-requirements.md](./active/frontend-requirements.md)
- 前端页面设计：[./active/frontend-page-spec.md](./active/frontend-page-spec.md)

## 下一步

1. 前端可先实现 admin 登录、session、权限守卫、员工、角色、发房方、房源只读总览。
2. 角色权限页面进入实现前，确认是否新增 `permission/list` 或由前端静态权限字典承接首版。
3. `launch_audit` 实现前补审核权限点，并和 publish 提交上架流程对齐。
4. 后续进入 `hs_lld_profile` 时，再单独补发房方资料维护需求。
