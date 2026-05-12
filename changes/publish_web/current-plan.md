# 出房 Web 端当前计划

本文是出房 Web 端的当前事实源。出房 Web 端最终会是独立前端仓库；当前仅因新仓库尚未初始化，临时在后台 Web 仓库新分支中开发。新仓库初始化后，直接迁移 `src/` 下相关代码并删除临时分支。

接口、字段和枚举口径以 [../../api/publish.md](../../api/publish.md) 和 [../../schema/db-design/v4/modules/house-master-data.md](../../schema/db-design/v4/modules/house-master-data.md) 为准。如果共享文档、旧代码、旧实现或历史聊天记录冲突，先读共享文档和当前 Go 后端实现，不要猜。

## 0. Contract Gate

未补齐 publish API DTO 前，不进入页面实现阶段。

必须先完成：

1. 补齐 [../../api/publish.md](../../api/publish.md) 中所有 action 的 request / response DTO。
2. 明确 publish API request / response 字段统一使用 `snake_case`。
3. 明确所有 `list_*` 接口是否分页，以及分页字段名。
4. 明确所有 `list_*` 返回 list item 字段。
5. 明确图片字段结构。
6. 明确 `create_*` / `update_*` / `*_detail` / `list_*` 返回完整 entity 还是只返回 ID。
7. 明确 `update_*_status` 请求字段和返回字段。
8. 明确错误码、空数据、空列表、资源不存在的口径。

前端 `types.ts` 必须从 [../../api/publish.md](../../api/publish.md) 契约实现，不得按 Go model、Mongo schema 或旧前端字段猜测生成。

### 0.1 DTO 命名

publish API DTO 统一使用 `snake_case`，与 miniapp API 保持一致。

使用：

```text
project_name
project_code
room_type_id
room_status
listing_facilities
```

不要使用：

```text
projectName
projectCode
roomTypeId
roomStatus
listingFacilities
```

约束：

- 后端不得直接把内部 Go model JSON 当作正式 publish API response。
- [../../api/publish.md](../../api/publish.md) 是前端 DTO 的唯一契约源。
- 前端 request / response DTO 使用 `snake_case`。
- 前端页面 ViewModel 可以按组件需要转换，但转换必须集中维护，不能散落在页面中。

### 0.2 图片字段

图片字段不能统一偷懒处理。

楼栋：

```ts
photos: string[]
```

房型和房间：

```ts
images: Array<{
  url: string
  tag?: string
}>
```

前端表单规则：

- 楼栋照片只录入 URL 列表。
- 房型、集中式房间、分散式房间图片录入 URL + 图片标签。
- `tag` 使用 HMD schema 中 `image_tags` 枚举。
- 表单提交枚举值，不提交中文文案。

## 1. 当前项目怎么运作

出房 Web 端使用与后台管理 Web 端一致的技术栈：

```text
Vue 3
Vite
TypeScript
SCSS
Element Plus
Pinia
Vue Router
Axios
```

当前临时开发位置：

```text
/Users/xinyue/VSCode/ws_2026/house-manager-web
```

临时共仓只是一日内过渡方案，不做长期共仓设计：

- 代码按未来独立出房 Web 仓库结构编写。
- 未来独立仓库结构优先参考当前后台 Web `feature_20260312` 分支的轻量架构。
- 不使用 `src/publish/` 作为最终目录结构；API module 叫 `publish` 不代表前端目录必须叫 `publish`。
- 不为了临时分支设计额外兼容层或迁出适配层。
- 明天新仓库初始化后，直接迁移 `src/` 下相关代码。
- 临时分支后续删除。

项目是全新项目，禁止兼容旧接口、旧字段、旧数据库、旧前端临时实现。

## 2. 0312 分支架构参考与推荐结构

### 2.1 0312 分支当前架构

当前后台 Web `feature_20260312` 分支是一个轻量 Vue3 后台 SPA，出房 Web 端应优先参考它的组织方式：

- `src/main.ts`：引入全局样式，创建 Vue app，注册 Element Plus、Pinia、router。
- `src/App.vue`：只承载 `<RouterView />`。
- `src/router/index.ts`：集中维护路由，使用 `createWebHistory`，路由 meta 驱动菜单和面包屑。
- `src/views/layout/`：维护 `Layout.vue`、`Header.vue`、`LeftMenu.vue`。
- `src/views/{module}/`：按页面模块放 Vue 页面。
- `src/api/{module}.ts`：封装接口调用。
- `src/model/{module}.ts`：维护请求、响应和页面使用的类型。
- `src/utils/request.ts`：Axios 实例、baseURL、拦截器。
- `src/utils/http.ts`：通用 HTTP 方法封装。
- `src/assets/css/index.scss`：统一引入 common 和页面样式。

出房 Web 端不需要引入 `src/app` / `src/shared` / `src/modules` 这类新分层，除非后续项目复杂度真的需要。

### 2.2 推荐独立仓库结构

推荐结构：

```text
src/
  App.vue
  main.ts

  router/
    index.ts

  views/
    layout/
      Layout.vue
      Header.vue
      LeftMenu.vue
    home/
      Home.vue
    centralized/
      project/
        ProjectList.vue
        ProjectDetail.vue
        ProjectForm.vue
      building/
        BuildingList.vue
        BuildingForm.vue
      roomType/
        RoomTypeList.vue
        RoomTypeForm.vue
      room/
        CentralizedRoomList.vue
        CentralizedRoomForm.vue
    decentralized/
      community/
        CommunityList.vue
        CommunityDetail.vue
        CommunityForm.vue
      room/
        DecentralizedRoomList.vue
        DecentralizedRoomForm.vue

  api/
    centralizedProject.ts
    building.ts
    roomType.ts
    centralizedRoom.ts
    decentralizedCommunity.ts
    decentralizedRoom.ts

  model/
    common.ts
    enums.ts
    centralizedProject.ts
    building.ts
    roomType.ts
    centralizedRoom.ts
    decentralizedCommunity.ts
    decentralizedRoom.ts

  utils/
    request.ts
    http.ts
    authToken.ts

  stores/
    auth.ts

  assets/
    css/
      index.scss
      common/
        common.scss
      centralized/
        project.scss
        building.scss
        roomType.scss
        room.scss
      decentralized/
        community.scss
        room.scss
```

第一期可以继续保持 0312 的轻量风格：

- 表单可以先和列表页放在同一个业务目录下，不需要提前抽很多公共组件。
- 只有 `PriceFields`、`ImageFields`、`FacilityFields` 这类真实复用且稳定的 UI，才考虑抽到 `views/common/` 或 `components/`。
- `api/` 和 `model/` 仍按业务对象拆分，避免一个巨大 `publish.ts`。
- `utils/request.ts` / `utils/http.ts` 参考 0312，但必须修正为 publish 当前接口口径：POST、`/api/v1`、`code === 0`、Bearer Token。

## 3. API 与前端模块关系

需要明确区分：

- API module：`publish`
- 前端页面模块：`views/centralized/project`、`views/centralized/building`、`views/centralized/roomType`、`views/centralized/room`、`views/decentralized/community`、`views/decentralized/room`
- 前端 API / model 文件：按业务对象命名，例如 `api/centralizedProject.ts`、`model/centralizedProject.ts`
- 仓库语义：出房 Web 独立仓库

API path 中出现 `publish`：

```text
POST /api/v1/publish/create_centralized_project
```

只应该体现在 API 封装里：

```ts
post('/publish/create_centralized_project', payload)
```

不应该决定前端目录叫 `src/publish/`。

## 4. 当前业务范围

第一期只接入 publish HMD 录入与维护链路。

对象范围：

- 集中式项目
- 集中式楼栋
- 集中式房型
- 集中式房间
- 分散式小区
- 分散式房间

接口统一走：

```text
POST /api/v1/publish/{action}
```

后端当前状态：

- publish 第一阶段 HMD 链路已通过真实服务联调。
- HMD 写入后会通过 `domain/hpd.Service.Apply(changes)` 刷新第一期小程序 HPD 展示快照。
- 受保护 publish 路由必须携带 `Authorization: Bearer <token>`。
- 分散式房间当前不写 `room_type_id`。

本期不做：

- 后台审核和运营管理。
- 小程序 C 端页面。
- HPD 发布内容编辑。
- 上架审核流。
- 房东端协作能力。
- 文件上传服务。
- 无后端接口支撑的统计、聚合、待办能力。

## 5. 通用 API 口径

### 5.1 请求层

请求层职责：

- 拼接 `VITE_API_BASE_URL + /api/v1 + 业务短路径`。
- 注入 `Authorization: Bearer <token>`。
- 解析 Go 后端统一响应。
- 以 `code === 0` 判定业务成功。
- HTTP 非 2xx 或 `code !== 0` 时读取 `error` 并统一提示。
- 遇到 Unauthorized 时清理本地 token，并提示重新设置 token 或重新登录。

统一响应：

```ts
interface ApiResponse<T> {
  code: number
  error: string
  data: T
}
```

API 层只写业务短路径，例如：

```text
/publish/create_centralized_project
/publish/list_centralized_projects
```

禁止：

- 页面直接拼完整 URL。
- 页面直接调用 Axios。
- API 文件重复写 `/api/v1`。
- 新增 REST 风格 `GET/PUT/DELETE` 主业务接口。
- 复用旧 house API。

### 5.2 页面路由与后端 API

前端页面路由可以按信息架构多级组织，例如：

```text
/centralized/projects/:project_id
/decentralized/communities/:community_id
```

后端 API 仍严格遵守：

```text
POST /api/v1/{module}/{action}
```

不要把前端页面路由风格带到后端 API 设计。

## 6. 鉴权口径

publish 路由当前需要 Bearer Token：

```text
Authorization: Bearer <token>
```

当前限制：

- 小程序 auth 不是 publish/admin 的通用登录入口。
- publish/admin auth 后续需要分别建立自己的端侧应用服务。

第一期前端处理方式：

- 请求层支持 Bearer Token。
- Token 从本地存储读取。
- 提供 dev-only 的开发联调 token 设置入口。
- 不复用小程序微信登录。
- 不写死 token。
- 不提交真实 token。
- 等 publish/admin auth 契约确认后，删除、隐藏或替换 dev token 入口。

建议开关：

```text
VITE_ENABLE_DEV_TOKEN_PANEL=true
```

dev token panel 约束：

- 仅开发联调用。
- 仅 dev 环境显示。
- 不作为正式登录方案。
- token 只保存在本地存储。
- Unauthorized 时清理 token，并提示重新设置 token。

## 7. 页面与路由规划

建议页面路由：

```text
/
/centralized/projects
/centralized/projects/:project_id
/centralized/projects/create
/centralized/projects/:project_id/edit
/centralized/buildings
/centralized/buildings/create
/centralized/buildings/:building_id/edit
/centralized/room-types
/centralized/room-types/create
/centralized/room-types/:room_type_id/edit
/centralized/rooms
/centralized/rooms/create
/centralized/rooms/:room_id/edit
/decentralized/communities
/decentralized/communities/:community_id
/decentralized/communities/create
/decentralized/communities/:community_id/edit
/decentralized/rooms
/decentralized/rooms/create
/decentralized/rooms/:room_id/edit
```

如果最终部署需要 `/publish` 前缀，应通过 Vite `base` 或路由 base 处理，不通过 `src/publish/` 目录表达。

## 8. Layout

出房端独立 Layout：

- 顶部：系统名、当前环境、dev token 状态入口。
- 左侧菜单：
  - 工作台
  - 集中式
    - 项目
    - 楼栋
    - 房型
    - 房间
  - 分散式
    - 小区
    - 房间
- 主区：列表、表单、详情、空状态。

## 9. 工作台

第一期工作台只做：

- 快捷入口。
- 静态空状态。
- 开发联调 token 状态。
- 常用模块入口。

第一期不做：

- 最近更新对象列表。
- 待补全数据提示。
- 统计卡片。
- 跨对象聚合信息。

除非后端 publish API 明确提供对应接口，否则前端不得硬造动态数据。

## 10. 模块计划

### 10.1 集中式项目

接口：

```text
create_centralized_project
centralized_project_detail
update_centralized_project
list_centralized_projects
```

页面能力：

- 项目列表。
- 创建项目。
- 编辑项目。
- 项目详情页作为楼栋、房型、房间的上下文入口。

字段以 HMD `hs_hmd_centralized` 为准，并以 publish API DTO 为最终契约：

```text
project_name
project_code
city
district
address_text
geo
brand_name
```

### 10.2 集中式楼栋

接口：

```text
create_building
building_detail
update_building
list_buildings_by_project
```

页面能力：

- 按项目筛选楼栋。
- 创建楼栋。
- 编辑楼栋。
- 楼栋详情页作为房型、房间筛选上下文。

字段以 HMD `hs_hmd_building` 为准，并以 publish API DTO 为最终契约：

```text
project_id
building_name
building_code
floor_total
manager_name
manager_phone
photos
listing_facilities
```

图片规则：

- `photos` 为 `string[]`。
- 只录入图片 URL 列表。

### 10.3 集中式房型

接口：

```text
create_room_type
room_type_detail
update_room_type
list_room_types_by_project
list_room_types_by_building
```

页面能力：

- 按项目或楼栋筛选房型。
- 创建房型。
- 编辑房型。
- 房型可作为集中式房间创建时的可选模板。

字段以 HMD `hs_hmd_room_type_centralized` 为准，并以 publish API DTO 为最终契约。

图片规则：

- `images` 为 `{ url, tag? }[]`。
- `tag` 使用 `image_tags` 枚举。

### 10.4 集中式房间

接口：

```text
create_centralized_room
centralized_room_detail
update_centralized_room
update_centralized_room_status
list_centralized_rooms_by_project
list_centralized_rooms_by_building
```

页面能力：

- 按项目或楼栋筛选房间。
- 创建房间。
- 编辑房间。
- 更新房态。
- 使用房型模板时可带 `room_type_id`。
- 可从房型模板带出部分字段，但提交 payload 必须以 publish API DTO 为准。

字段以 HMD `hs_hmd_room_centralized` 为准，并以 publish API DTO 为最终契约。

图片规则：

- `images` 为 `{ url, tag? }[]`。
- `tag` 使用 `image_tags` 枚举。

### 10.5 分散式小区

接口：

```text
create_decentralized_community
decentralized_community_detail
update_decentralized_community
list_decentralized_communities
list_decentralized_communities_by_city
list_decentralized_communities_by_district
```

页面能力：

- 小区列表。
- 创建小区。
- 编辑小区。
- 小区详情页作为分散式房间入口。

字段以 HMD `hs_hmd_decentralized` 为准，并以 publish API DTO 为最终契约：

```text
community_name
city
district
biz_area
address_text
geo
subway_station
```

### 10.6 分散式房间

接口：

```text
create_decentralized_room
decentralized_room_detail
update_decentralized_room
update_decentralized_room_status
list_decentralized_rooms_by_community
```

页面能力：

- 按小区筛选房间。
- 创建房间。
- 编辑房间。
- 更新房态。

约束：

- 分散式房间当前没有房型模型。
- 分散式房间 create/update request type 中不得出现 `room_type_id`。
- 分散式房间 API payload builder 中不得出现 `room_type_id`。
- 集中式房间可以出现 `room_type_id`。

字段以 HMD `hs_hmd_room_decentralized` 为准，并以 publish API DTO 为最终契约。

图片规则：

- `images` 为 `{ url, tag? }[]`。
- `tag` 使用 `image_tags` 枚举。

## 11. 房间表单规则

集中式房间和分散式房间不要强行抽象成同一个 `RoomForm`。

集中式房间上下文：

- `project_id`
- `building_id`
- `room_type_id` 可选
- 可以从房型模板带出部分字段

分散式房间上下文：

- `decentralized_id` 或契约确认后的 `community_id`
- 当前没有 `room_type_id`
- 不依赖房型模板

必须使用：

- `CentralizedRoomForm`
- `DecentralizedRoomForm`

可以共享小组件：

- `PriceFields`
- `ImageFields`
- `FacilityFields`
- `RoomStatusSelect`
- `RentModeSelect`
- `DecorationSelect`

禁止：

- 做一个包含大量 `if centralized` / `if decentralized` 的万能 `RoomForm`。
- 分散式房间请求包含 `room_type_id`。
- 为了复用表单保留无效字段。

payload builder 必须分开维护。

## 12. 枚举与表单规则

枚举值以 [../../schema/db-design/v4/modules/house-master-data.md](../../schema/db-design/v4/modules/house-master-data.md) 为准。

当前需要前端常量化的枚举：

```text
rent_mode
room_status
decoration_level
payment_cycle
agency_fee_mode
viewing_time_rule
start_rent_rule
orientation
listing_facilities
room_facilities
image_tags
```

规则：

- 不在前端自造枚举值。
- 表单提交使用 schema 中的枚举值，不提交中文文案。
- `0`、`""`、`null`、`undefined` 表示未指定，不解释成有效业务状态。
- 页面展示默认值时不得直接展示 `0` 或空字符串，应展示为空态或“未填写”。

## 13. 修改顺序

### 13.1 Contract Gate

必须先完成：

1. 补齐 [../../api/publish.md](../../api/publish.md) 所有 request / response DTO。
2. 明确 publish API 字段统一 `snake_case`。
3. 明确 list 是否分页。
4. 明确 list item 字段。
5. 明确图片字段结构。
6. 明确 create/update/detail/list 返回完整 entity 还是 ID。
7. 明确 status 更新字段。
8. 明确错误码和空数据行为。

未完成本阶段前，不进入页面实现。

### 13.2 Frontend Skeleton

目标：

- 按 0312 分支轻量后台架构建立未来独立出房 Web 仓库结构。

任务：

1. 建立 `src/main.ts`、`src/App.vue`、`src/router/index.ts`。
2. 建立 `src/views/layout/Layout.vue`、`Header.vue`、`LeftMenu.vue`。
3. 建立 `src/views/home/Home.vue`。
4. 建立 `src/api/*` 和 `src/model/*` 的业务对象文件。
5. 建立 `src/utils/request.ts`、`src/utils/http.ts`、`src/utils/authToken.ts`。
6. 建立 `ApiResponse` 类型和 publish 请求成功判定。
7. 建 dev-only token panel。
8. 建枚举常量和展示 mapper。
9. 建 `src/assets/css/index.scss`、`common` 和业务模块样式入口。

当前落地状态（2026-05-12）：

- 已建立 `main.ts`、`App.vue`、路由、Layout、工作台、第一期模块菜单和占位路由。
- 已建立按业务对象拆分的 `api/*` 与 `model/*`，DTO 使用 `snake_case`。
- 已建立 `utils/request.ts`、`utils/http.ts`、`utils/authToken.ts`，请求路径为 `/api/v1 + /publish/{action}`，业务成功以 `code === 0` 判定。
- 已建立 dev-only token panel；仅当 `VITE_ENABLE_DEV_TOKEN_PANEL=true` 且处于 dev 模式时展示。
- 已建立枚举常量、展示 mapper 和基础全局样式。
- 已补齐集中式楼栋、房型、房间以及分散式小区、房间的业务页面目录和路由落点；未实现的页面暂保留模块占位内容。
- `npm run build` 已通过；Vite 仍有首包大于 500 kB 的提示，当前阶段暂不拆包。

下一步进入 `13.3 Centralized MVP`，先实现集中式项目 list/create/detail/update，再接楼栋、房型、房间链路。

### 13.3 Centralized MVP

目标：

- 跑通集中式项目、楼栋、房型、房间主链路。

任务：

1. `centralized-project` list/create/detail/update。
2. `building` list/create/update。
3. `room-type` list/create/update。
4. `centralized-room` list/create/update/status。
5. 级联选择：project -> building -> room type / room。

当前落地状态（2026-05-12）：

- 已完成 `centralized-project` list/create/detail/update 页面。
- 项目列表要求输入 `city` 后调用 `list_centralized_projects`，不在前端伪造动态数据。
- 创建/编辑表单按 publish DTO 提交 `snake_case` 字段，`project_code` 仅创建时可填。
- 项目详情页提供楼栋、房型、房间上下文入口。
- `npm run build` 已通过；下一步继续实现 `building` list/create/update。

### 13.4 Decentralized MVP

目标：

- 跑通分散式小区和房间主链路。

任务：

1. `decentralized-community` list/create/detail/update。
2. `decentralized-room` list/create/update/status。
3. 级联选择：city/district -> community -> room。
4. 确认 decentralized-room payload 不含 `room_type_id`。

### 13.5 E2E

目标：

- 完成真实后端 E2E。

任务：

1. 使用真实 Bearer Token。
2. 验证 create/detail/list/update/status 全链路。
3. 验证 HMD 写入后 HPD projector 不报错。
4. 跑 `npm run build`。
5. 记录 DTO 偏差并同步 shared-docs。

## 14. 验收标准

### 14.1 自动检查

需要通过：

```text
npm run build
```

出房端代码中不应出现：

```text
uni.request
wx.login
GET /api
PUT /api
DELETE /api
```

分散式房间专项检查：

- 分散式房间 create/update request type 中不得出现 `room_type_id`。
- 分散式房间 API payload builder 中不得出现 `room_type_id`。
- 集中式房间允许出现 `room_type_id`。

### 14.2 E2E

集中式链路：

1. 创建项目。
2. 查询项目列表。
3. 查看项目详情。
4. 创建楼栋。
5. 创建房型。
6. 创建集中式房间。
7. 更新集中式房间状态。

分散式链路：

1. 创建小区。
2. 查询小区列表。
3. 查看小区详情。
4. 创建分散式房间。
5. 更新分散式房间状态。

### 14.3 后端日志期望

应出现：

```text
POST /api/v1/publish/create_centralized_project
POST /api/v1/publish/list_centralized_projects
POST /api/v1/publish/create_building
POST /api/v1/publish/create_room_type
POST /api/v1/publish/create_centralized_room
POST /api/v1/publish/update_centralized_room_status
POST /api/v1/publish/create_decentralized_community
POST /api/v1/publish/create_decentralized_room
POST /api/v1/publish/update_decentralized_room_status
```

不应出现：

```text
GET /api/v1/publish/*
PUT /api/v1/publish/*
DELETE /api/v1/publish/*
POST /api/v1/house/create
```

## 15. 当前重要文档

日常只需要先读这些：

1. [../../README.md](../../README.md)：项目文档入口和阅读路径。
2. [../../frontend/publish.md](../../frontend/publish.md)：出房前端长期约定。
3. [../../api/publish.md](../../api/publish.md)：出房端 API 契约。
4. [../../product/publish.md](../../product/publish.md)：出房系统产品范围。
5. [../../flows/publish-listing.md](../../flows/publish-listing.md)：发房到 HMD/HPD 的链路。
6. [../../schema/db-design/v4/modules/house-master-data.md](../../schema/db-design/v4/modules/house-master-data.md)：HMD 字段和枚举。
7. [../go_backend/current-plan.md](../go_backend/current-plan.md)：Go 后端当前进度。

如果这些文档和历史聊天记录冲突，以这些文档为准。

## 16. 文档维护规则

- 出房 Web 端当前状态只维护本文。
- 做完任务后更新本文的“当前业务范围 / 修改顺序 / 验收标准”。
- 长期有效的前端约定写入 [../../frontend/publish.md](../../frontend/publish.md)。
- 长期有效的接口口径写入 [../../api/publish.md](../../api/publish.md)。
- 字段、枚举、索引变化写入 [../../schema/db-design/v4/modules/house-master-data.md](../../schema/db-design/v4/modules/house-master-data.md)。
- 已废弃接口、旧路径、旧字段说明应删除，不在 changes 里长期堆积。
