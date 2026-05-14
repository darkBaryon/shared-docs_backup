# 页面工作台化收敛计划

状态：已完成实现与本地验证。

## 背景

当前出房 Web MVP 已按业务对象补齐页面：

- 集中式项目、楼栋、房型、房间。
- 分散式小区、房间。

问题是页面入口太散。用户维护一个项目时，需要在楼栋、房型、房间多个菜单之间跳转；维护一个小区时，也要离开小区上下文去改房间。

本次目标不是合并业务模型，也不是做万能配置页，而是收敛用户可见页面入口。

## 目标形态

用户可见主入口收敛为：

```text
集中式
  项目

分散式
  小区
```

集中式项目详情变成项目工作台：

```text
项目信息
  楼栋 tab
  房型 tab
  房间 tab
```

分散式小区详情变成小区工作台：

```text
小区信息
  房间列表
```

新增和编辑子对象使用 drawer，不再离开工作台页面。

## 非目标

- 不做万能 `EntityPage` / `DynamicForm`。
- 不把集中式房间和分散式房间合并成一个带大量 `if` 的通用组件。
- 不做旧路由兼容设计。
- 不新增后端接口。
- 不改变 `api/*` 和 `model/*` 的业务对象拆分。
- 不改变 publish API 路径。

## 路由和菜单

保留页面路由：

```text
/
/centralized/projects
/centralized/projects/create
/centralized/projects/:project_id
/centralized/projects/:project_id/edit
/decentralized/communities
/decentralized/communities/create
/decentralized/communities/:community_id
/decentralized/communities/:community_id/edit
```

移除独立子模块页面路由：

```text
/centralized/buildings*
/centralized/room-types*
/centralized/rooms*
/decentralized/rooms*
```

子模块维护入口全部从项目或小区工作台进入。

## 文件计划

修改：

```text
src/router/index.ts
src/views/centralized/project/ProjectDetail.vue
src/views/decentralized/community/CommunityDetail.vue
```

新增或改造：

```text
src/views/centralized/project/components/ProjectBuildingsPanel.vue
src/views/centralized/project/components/ProjectRoomTypesPanel.vue
src/views/centralized/project/components/ProjectRoomsPanel.vue
src/views/decentralized/community/components/CommunityRoomsPanel.vue

src/views/centralized/building/BuildingEditor.vue
src/views/centralized/roomType/RoomTypeEditor.vue
src/views/centralized/room/CentralizedRoomEditor.vue
src/views/decentralized/room/DecentralizedRoomEditor.vue
```

完成抽取后删除不再被 router 引用的旧独立子模块页面：

```text
src/views/centralized/building/BuildingList.vue
src/views/centralized/building/BuildingForm.vue
src/views/centralized/roomType/RoomTypeList.vue
src/views/centralized/roomType/RoomTypeForm.vue
src/views/centralized/room/CentralizedRoomList.vue
src/views/centralized/room/CentralizedRoomForm.vue
src/views/decentralized/room/DecentralizedRoomList.vue
src/views/decentralized/room/DecentralizedRoomForm.vue
```

## 交互约束

- Drawer 宽度桌面约 720px，窄屏 100%。
- Drawer 标题根据 create/edit 切换。
- 保存中禁用保存按钮。
- 保存成功后关闭 drawer 并刷新当前列表。
- Tab 可按需懒加载，不必一次性加载全部子模块。
- 分散式房间 request 和页面 payload 不得出现 `room_type_id`。

## API 约束

继续使用现有业务 API 封装：

```text
api/centralizedProject.ts
api/building.ts
api/roomType.ts
api/centralizedRoom.ts
api/decentralizedCommunity.ts
api/decentralizedRoom.ts
```

禁止：

- 页面直接调用 axios。
- 页面拼完整 `/api/v1` URL。
- 新增 `/api/v1/publish/*`。
- 新增主业务 GET / PUT / DELETE。
- 为旧页面路由增加兼容 API 或字段。

## 推荐顺序

1. 改 `router/index.ts` 菜单，只保留项目和小区入口。
2. 抽 `BuildingEditor.vue`，接入 `ProjectBuildingsPanel.vue`。
3. 抽 `RoomTypeEditor.vue`，接入 `ProjectRoomTypesPanel.vue`。
4. 抽 `CentralizedRoomEditor.vue`，接入 `ProjectRoomsPanel.vue`。
5. 抽 `DecentralizedRoomEditor.vue`，接入 `CommunityRoomsPanel.vue`。
6. 删除不再被 router 引用的旧独立子模块页面。
7. 更新本 plan 状态和 [../status.md](../status.md)。

## 验证

命令：

```text
npm run lint
npm run build
```

检查：

```text
rg "uni\\.request|wx\\.login|GET /api|PUT /api|DELETE /api|/api/v1/publish|/publish/|PUBLISH_API_PREFIX|method:\\s*'get'|method:\\s*'put'|method:\\s*'delete'" src
rg "room_type_id" src/model/decentralizedRoom.ts src/api/decentralizedRoom.ts src/views/decentralized
```

验收：

- 左侧菜单只展示项目和小区。
- 项目详情能维护楼栋、房型、集中式房间。
- 小区详情能维护分散式房间。
- drawer 保存后列表刷新。
- 分散式房间 payload 不包含 `room_type_id`。
- build/lint 通过。

## 实现结果

- 左侧菜单和首页入口已收敛为集中式项目、分散式小区。
- 集中式项目详情已改为工作台，包含项目信息、楼栋 tab、房型 tab、房间 tab。
- 分散式小区详情已改为工作台，包含小区信息和房间列表。
- 楼栋、房型、集中式房间、分散式房间新增/编辑均改为 drawer，不再通过独立子模块路由进入。
- 已删除不再被 router 引用的旧独立子模块 list/form 页面。

## 本地验证结果

已执行：

```text
npm run lint
npm run build
rg "uni\\.request|wx\\.login|GET /api|PUT /api|DELETE /api|/api/v1/publish|/publish/|PUBLISH_API_PREFIX|method:\\s*'get'|method:\\s*'put'|method:\\s*'delete'" src
rg "room_type_id" src/model/decentralizedRoom.ts src/api/decentralizedRoom.ts src/views/decentralized
rg "/centralized/buildings|/centralized/room-types|/centralized/rooms|/decentralized/rooms" src
```

结果：

- lint 通过。
- build 通过；仅有 Vite chunk size 提示。
- 三个 `rg` 检查均无命中。
- 真实后端和 Bearer Token E2E 未执行，仍待后续联调验证。
