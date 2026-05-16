# Admin Web Status

## 当前阶段

管理后台处于产品与工程设计阶段，尚未开始正式实现。

当前不能直接开工的原因：

- 账号与身份 schema 已初步完成，但 Go model/repo 尚未实现
- 上架审核单状态流未定

## 当前结论

- 管理后台是独立前端，不是 `publish` 的增强版。
- 管理后台服务对象是平台员工，核心职责是审核、运营、分发、跟进、组织与权限管理。
- 管理后台后端独立走：

```text
handler/v1/admin/*
  -> service/admin/*
```

- 可以复用 `domain/*` 和 `repository/*`，但不复用 `service/miniapp/*` 或 `service/publish/*`。
- 发房方接口层 `provider_id` 已确定为 `hs_lld_landlord._id`。
- 当前阶段发房方只实现 `hs_lld_landlord + hs_lld_auth`，`hs_lld_profile` 暂不进入实现。

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

## 下一步

1. 继续补 admin staff / landlord 相关 Go model 与 repository 计划。
2. 再固化 admin API 第一批模块边界。
3. 审核单状态流待业务明确后单独设计。
4. schema 拍板后再进入前后端实现。
