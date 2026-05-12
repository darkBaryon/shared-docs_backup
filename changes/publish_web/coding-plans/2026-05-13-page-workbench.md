# 2026-05-13 页面工作台化收敛 Coding Plan

状态：待 review，未开始实现。

## 1. 背景

当前出房 Web MVP 已按业务对象补齐页面：

- 集中式项目
- 集中式楼栋
- 集中式房型
- 集中式房间
- 分散式小区
- 分散式房间

问题是这些页面的列表、筛选、创建、编辑体验高度相似。用户维护一个项目时，需要在楼栋、房型、房间多个菜单之间跳转，操作路径偏散。

本次目标不是合并业务模型，也不是做万能配置页，而是收敛用户可见页面入口。

## 2. 目标

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

## 3. 非目标

本次不做：

- 不做万能 `EntityPage` / `DynamicForm`。
- 不把集中式房间和分散式房间合并成一个带大量 `if` 的通用 `RoomForm`。
- 不做旧路由兼容设计；新项目尚未发布，不需要保留旧页面入口。
- 不新增后端接口。
- 不改变 `api/*` 和 `model/*` 的业务对象拆分。
- 不改变 publish API 路径：仍为 `/api/v1/{business_module}/{action}`。

## 4. 信息架构

### 4.1 左侧菜单

调整前：

```text
工作台
集中式
  项目
  楼栋
  房型
  房间
分散式
  小区
  房间
```

调整后：

```text
工作台
集中式
  项目
分散式
  小区
```

### 4.2 路由

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
/centralized/buildings
/centralized/buildings/create
/centralized/buildings/:building_id/edit
/centralized/room-types
/centralized/room-types/create
/centralized/room-types/:room_type_id/edit
/centralized/rooms
/centralized/rooms/create
/centralized/rooms/:room_id/edit
/decentralized/rooms
/decentralized/rooms/create
/decentralized/rooms/:room_id/edit
```

说明：

- 本项目是全新项目，不为当前临时页面路由做兼容保留。
- 子模块维护入口全部从项目或小区工作台进入。

## 5. 文件改造计划

### 5.1 路由和菜单

修改：

```text
src/router/index.ts
```

动作：

- 删除楼栋、房型、集中式房间、分散式房间的菜单项。
- 删除这些子模块的独立页面路由。
- 保留项目和小区相关路由。

### 5.2 集中式项目工作台

修改：

```text
src/views/centralized/project/ProjectDetail.vue
```

新增：

```text
src/views/centralized/project/components/ProjectBuildingsPanel.vue
src/views/centralized/project/components/ProjectRoomTypesPanel.vue
src/views/centralized/project/components/ProjectRoomsPanel.vue
```

动作：

- `ProjectDetail.vue` 顶部展示项目信息摘要和编辑按钮。
- 下方使用 `el-tabs`：
  - `building`
  - `roomType`
  - `room`
- 每个 panel 接收 `project_id`。
- panel 内负责列表加载、空态、创建按钮、编辑按钮和 drawer 状态。
- drawer 保存成功后关闭并刷新当前列表。

### 5.3 集中式子模块编辑组件

新增或改造：

```text
src/views/centralized/building/BuildingEditor.vue
src/views/centralized/roomType/RoomTypeEditor.vue
src/views/centralized/room/CentralizedRoomEditor.vue
```

动作：

- 从原 `BuildingForm.vue`、`RoomTypeForm.vue`、`CentralizedRoomForm.vue` 中抽出表单主体。
- 编辑组件只接收必要上下文 props，不直接依赖页面路由：
  - `BuildingEditor`: `project_id`, optional `building_id`
  - `RoomTypeEditor`: `project_id`, optional `building_id`, optional `room_type_id`
  - `CentralizedRoomEditor`: `project_id`, optional `building_id`, optional `room_type_id`, optional `room_id`
- 编辑组件通过 emit 通知：
  - `saved`
  - `cancel`
- payload builder 仍按各自业务对象维护，不做统一万能 builder。

### 5.4 分散式小区工作台

修改：

```text
src/views/decentralized/community/CommunityDetail.vue
```

新增：

```text
src/views/decentralized/community/components/CommunityRoomsPanel.vue
```

动作：

- `CommunityDetail.vue` 顶部展示小区信息摘要和编辑按钮。
- 下方展示分散式房间列表。
- 新增/编辑房间打开 drawer。
- 保存成功后刷新房间列表。

### 5.5 分散式房间编辑组件

新增或改造：

```text
src/views/decentralized/room/DecentralizedRoomEditor.vue
```

动作：

- 从原 `DecentralizedRoomForm.vue` 中抽出表单主体。
- 组件接收：
  - `decentralized_id`
  - optional `room_id`
- request 和页面 payload 继续禁止出现 `room_type_id`。
- emit `saved` / `cancel`。

### 5.6 旧页面文件处理

当前独立页可在完成组件抽取后删除：

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

删除条件：

- 对应 panel 和 editor 已完成。
- router 不再引用旧页面。
- build 和 lint 通过。

## 6. 交互细节

### 6.1 Drawer

统一使用右侧 drawer：

- 宽度：桌面 720px 左右，窄屏使用 100%。
- 标题根据 create/edit 切换。
- 底部固定取消/保存按钮。
- 保存中禁用保存按钮。
- 保存成功后关闭 drawer 并刷新列表。

### 6.2 列表刷新

- panel 首次 mounted 时加载列表。
- 切换 tab 时，如果该 tab 未加载过，则加载一次。
- 保存后强制刷新当前 panel。
- 不做前端伪造动态统计。

### 6.3 上下文传递

集中式：

- 楼栋必须带 `project_id`。
- 房型必须带 `project_id`，可选 `building_id`。
- 集中式房间必须带 `project_id`，可选 `building_id` / `room_type_id`。

分散式：

- 分散式房间必须带 `decentralized_id`。
- 分散式房间不得出现 `room_type_id`。

## 7. API 约束

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

## 8. 推荐实施顺序

1. 改 `router/index.ts` 菜单，只保留项目和小区入口。
2. 为集中式楼栋抽 `BuildingEditor.vue`。
3. 新建 `ProjectBuildingsPanel.vue` 并接入 `ProjectDetail.vue`。
4. 为集中式房型抽 `RoomTypeEditor.vue`。
5. 新建 `ProjectRoomTypesPanel.vue` 并接入 `ProjectDetail.vue`。
6. 为集中式房间抽 `CentralizedRoomEditor.vue`。
7. 新建 `ProjectRoomsPanel.vue` 并接入 `ProjectDetail.vue`。
8. 为分散式房间抽 `DecentralizedRoomEditor.vue`。
9. 新建 `CommunityRoomsPanel.vue` 并接入 `CommunityDetail.vue`。
10. 删除不再被 router 引用的旧独立子模块页面。
11. 更新本 plan 状态和 `current-plan.md`。

## 9. 验证

命令：

```text
npm run lint
npm run build
```

grep：

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

## 10. 风险和注意事项

- 抽表单时不要把集中式/分散式房间合并成一个组件。
- 不要因为 drawer 复用而新增无效字段。
- 旧独立页删除前要确认 router 引用已清理。
- 项目详情一次性加载三个 tab 可能增加请求量；建议按 tab 懒加载。
- 如真实后端字段与 shared_docs 不一致，先记录 DTO 偏差并更新契约，不在前端猜字段。
