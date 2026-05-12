# 出房 Web 当前计划

本文只保留当前状态和下一步入口。历史长计划已归档到 [archive/2026-05-12-current-plan-full.md](archive/2026-05-12-current-plan-full.md)。

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

## 当前问题

当前 MVP 页面按业务对象拆成多个独立列表和表单页，工程边界清楚，但用户体验像复制了多套后台页面。

更合理的产品形态应收敛为：

- 集中式：项目列表 + 项目工作台。
- 分散式：小区列表 + 小区工作台。
- 楼栋、房型、房间作为工作台里的子模块维护，不再作为一等菜单页。

## 待 Review Coding Plan

- [2026-05-13 页面工作台化收敛](coding-plans/2026-05-13-page-workbench.md)

review 通过前，不进入代码实现。

## 下一步

1. review 页面工作台化 coding plan。
2. 确认后再改前端代码。
3. 实现后执行：

```text
npm run lint
npm run build
```

4. 使用真实后端和 Bearer Token 做 E2E。

## 文档维护

本目录结构说明见 [README.md](README.md)。
