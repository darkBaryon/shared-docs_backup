# 出房 Web 工作流状态

本文只保留出房 Web 当前状态和任务入口。具体 coding plan 放在 `active/`，review report 放在 `reviews/`。

## 当前事实

- 出房 Web 端当前临时开发仓库：`/Users/xinyue/VSCode/ws_2026/house-manager-web`
- 当前前端分支：`feat_20260512_publish_web`
- 项目是全新项目，不做旧接口、旧字段、旧数据库、旧前端兼容。
- 接口契约以 [../../api/publish.md](../../api/publish.md) 为准。
- 前端长期约定以 [../../frontend/publish.md](../../frontend/publish.md) 为准。

## 已完成

- 已建立轻量 Vue 3 / Vite / TypeScript / Element Plus 架构。
- 已建立 `api/*`、`model/*`，按业务对象拆分。
- 请求层已统一 `/api/v1 + /{business_module}/{action}`。
- 已完成集中式项目、楼栋、房型、房间以及分散式小区、房间的 MVP 页面。
- 已确认分散式房间 request 和页面 payload 不包含 `room_type_id`。
- 已完成页面工作台化收敛：
  - 用户可见主入口收敛为集中式项目、分散式小区。
  - 集中式项目详情已包含项目信息、楼栋、房型、房间工作台。
  - 分散式小区详情已包含小区信息、房间列表工作台。
  - 子对象新增/编辑已改为 drawer，不再离开工作台页面。
  - 旧独立楼栋、房型、房间子模块路由和页面已移除。
- 页面工作台化已完成本地 lint/build 和静态约束检查。

## 当前状态

- 页面入口和工作台形态已按 [页面工作台化收敛](./active/page-workbench.md) 完成。
- 当前未执行真实后端和 Bearer Token E2E，仍需在联调环境补测主要创建/编辑/房态更新流程。

## 当前任务

- 配合 [Publish Auth 与数据作用域计划](../backend-go/active/publish-auth-data-scope.md) 接入正式登录态和账号作用域；阶段 2 前端登录态接入已实现并 review 通过，待阶段 3 后端 HPD entrust relation。
- 页面工作台化收敛进入最终 review。
- 等待真实后端 E2E 验证。

## 下一步

1. Review 页面工作台化实现。
2. 分阶段接入 publish auth、session 恢复和账号作用域。
3. 使用真实后端和 Bearer Token 做 E2E。
4. 根据 E2E 结果修正联调问题。
