# publish-web src 目录重构清单

状态：草案，待按阶段执行。

## 目标

基于 `house-manager-web` 当前仓库真实 `src` 结构，做一次仅面向当前项目的前端目录收敛。

本清单只服务当前确认需求，不做任何兼容性设计，不为历史结构保留过渡目录，不为旧页面、旧命名、旧样式布局做适配。

## 用户已确认的结构约束

1. 目录主轴保持为：

```text
views/
components/
api/
model/
utils/
stores/
router/
assets/css/
```

2. `vue` 文件与 `scss` 文件分离。
3. `scss` 统一放在 `src/assets/css/*`。
4. 页面按业务组织，不做通用 `features`、`service`、`repository` 分层。
5. `views/common` 必须清理。
6. 不碰后端，不引入兼容层，不为旧结构保留双写。

## 当前 src 现状

当前仓库真实文件如下：

```text
src/
  App.vue
  main.ts

  router/
    index.ts

  stores/
    auth.ts

  utils/
    authToken.ts
    http.ts
    request.ts

  api/
    building.ts
    centralizedProject.ts
    centralizedRoom.ts
    decentralizedCommunity.ts
    decentralizedRoom.ts
    publishAuth.ts
    roomType.ts

  model/
    building.ts
    centralizedProject.ts
    centralizedRoom.ts
    common.ts
    decentralizedCommunity.ts
    decentralizedRoom.ts
    enums.ts
    publishAuth.ts
    roomType.ts

  views/
    auth/
      DevTokenDialog.vue
      LoginView.vue

    common/
      ModulePlaceholder.vue
      TaggedImageFields.vue
      formHelpers.ts
      listFilterStorage.ts

    home/
      Home.vue

    layout/
      Header.vue
      Layout.vue
      LeftMenu.vue

    centralized/
      building/
        BuildingEditor.vue
      project/
        ProjectDetail.vue
        ProjectForm.vue
        ProjectList.vue
        components/
          ProjectBuildingsPanel.vue
          ProjectRoomTypesPanel.vue
          ProjectRoomsPanel.vue
      room/
        CentralizedRoomEditor.vue
      roomType/
        RoomTypeEditor.vue

    decentralized/
      community/
        CommunityDetail.vue
        CommunityForm.vue
        CommunityList.vue
        components/
          CommunityRoomsPanel.vue
      room/
        DecentralizedRoomEditor.vue

  assets/
    css/
      index.scss
      common/
        common.scss
        publish-workbench.scss
```

## 当前问题

### 1. `views/common` 已经变成杂物层

当前目录里同时混放了：

- UI 组件：`TaggedImageFields.vue`
- 路由/展示/清洗工具：`formHelpers.ts`
- 本地筛选缓存：`listFilterStorage.ts`
- 占位页：`ModulePlaceholder.vue`

这些东西不属于同一层，继续堆会越来越脏。

### 2. 多个 editor 目录位置不对

以下文件并不是一级业务页面，而是项目详情或小区详情里的编辑抽屉：

- `views/centralized/building/BuildingEditor.vue`
- `views/centralized/roomType/RoomTypeEditor.vue`
- `views/centralized/room/CentralizedRoomEditor.vue`
- `views/decentralized/room/DecentralizedRoomEditor.vue`

它们继续挂在独立目录里，会让目录看起来像还有独立楼栋/房型/房间模块，和当前页面工作台方向冲突。

### 3. 页面命名还停留在旧阶段

当前页面文件：

- `ProjectList.vue`
- `ProjectForm.vue`
- `ProjectDetail.vue`
- `CommunityList.vue`
- `CommunityForm.vue`
- `CommunityDetail.vue`
- `LoginView.vue`
- `Home.vue`

已经都是路由页，但命名还不够明确。

### 4. 样式组织过粗

当前只有：

- `assets/css/common/common.scss`
- `assets/css/common/publish-workbench.scss`

样式已经被合并得过大，不利于后续按页面维护；同时也不应该反过来把每个 panel、drawer 都默认拆成一份独立 scss。

### 5. `api`、`model`、`utils` 仍然过于平铺

当前规模还不需要重型分层，但已经值得做按业务域的二级目录收拢。

## 目标目录

本轮重构后的目标结构如下：

```text
src/
  App.vue
  main.ts

  router/
    index.ts

  stores/
    auth.ts

  api/
    auth/
      publishAuth.ts
    centralized/
      building.ts
      project.ts
      room.ts
      roomType.ts
    decentralized/
      community.ts
      room.ts

  model/
    auth/
      publishAuth.ts
    centralized/
      building.ts
      project.ts
      room.ts
      roomType.ts
    decentralized/
      community.ts
      room.ts
    shared/
      common.ts
      enums.ts

  utils/
    auth/
      authToken.ts
    http/
      http.ts
      request.ts
    format/
      display.ts
      money.ts
      time.ts
    route/
      query.ts
    normalize/
      taggedImage.ts
      value.ts
    storage/
      listFilterStorage.ts

  views/
    auth/
      LoginPage.vue
    home/
      HomePage.vue
    layout/
      AppHeader.vue
      AppLayout.vue
      AppSidebar.vue
    centralized/
      project/
        ProjectListPage.vue
        ProjectFormPage.vue
        ProjectDetailPage.vue
    decentralized/
      community/
        CommunityListPage.vue
        CommunityFormPage.vue
        CommunityDetailPage.vue

  components/
    auth/
      DevTokenDialog.vue
    shared/
      TaggedImageFields.vue
    centralized/
      project/
        BuildingEditorDrawer.vue
        ProjectBuildingsPanel.vue
        ProjectRoomTypesPanel.vue
        ProjectRoomsPanel.vue
        RoomTypeEditorDrawer.vue
        CentralizedRoomEditorDrawer.vue
    decentralized/
      community/
        CommunityRoomsPanel.vue
        DecentralizedRoomEditorDrawer.vue

  assets/
    css/
      index.scss
      layout/
        app-header.scss
        app-layout.scss
        app-sidebar.scss
      auth/
        login-page.scss
      home/
        home-page.scss
      centralized/
        project-list-page.scss
        project-form-page.scss
        project-detail-page.scss
      decentralized/
        community-list-page.scss
        community-form-page.scss
        community-detail-page.scss
      shared/
        tagged-image-fields.scss
```

样式策略说明：

- 页面样式以页面级 scss 为主。
- layout 作为全局壳层，可以单独维护 `layout/*.scss`。
- panel、drawer、普通业务组件默认不单独拆 scss，优先归入所属页面 scss。
- 只有真正跨页面复用且样式本身复杂的组件，才单独放一份 scss，例如 `TaggedImageFields.vue`。

## 逐文件 rename / move 清单

### 1. views 页面重命名

```text
src/views/auth/LoginView.vue
-> src/views/auth/LoginPage.vue

src/views/home/Home.vue
-> src/views/home/HomePage.vue

src/views/layout/Header.vue
-> src/views/layout/AppHeader.vue

src/views/layout/Layout.vue
-> src/views/layout/AppLayout.vue

src/views/layout/LeftMenu.vue
-> src/views/layout/AppSidebar.vue

src/views/centralized/project/ProjectList.vue
-> src/views/centralized/project/ProjectListPage.vue

src/views/centralized/project/ProjectForm.vue
-> src/views/centralized/project/ProjectFormPage.vue

src/views/centralized/project/ProjectDetail.vue
-> src/views/centralized/project/ProjectDetailPage.vue

src/views/decentralized/community/CommunityList.vue
-> src/views/decentralized/community/CommunityListPage.vue

src/views/decentralized/community/CommunityForm.vue
-> src/views/decentralized/community/CommunityFormPage.vue

src/views/decentralized/community/CommunityDetail.vue
-> src/views/decentralized/community/CommunityDetailPage.vue
```

### 2. 业务组件收拢到 components

```text
src/views/auth/DevTokenDialog.vue
-> src/components/auth/DevTokenDialog.vue

src/views/common/TaggedImageFields.vue
-> src/components/shared/TaggedImageFields.vue

src/views/centralized/project/components/ProjectBuildingsPanel.vue
-> src/components/centralized/project/ProjectBuildingsPanel.vue

src/views/centralized/project/components/ProjectRoomTypesPanel.vue
-> src/components/centralized/project/ProjectRoomTypesPanel.vue

src/views/centralized/project/components/ProjectRoomsPanel.vue
-> src/components/centralized/project/ProjectRoomsPanel.vue

src/views/decentralized/community/components/CommunityRoomsPanel.vue
-> src/components/decentralized/community/CommunityRoomsPanel.vue
```

### 3. editor 文件按所属工作台收拢

```text
src/views/centralized/building/BuildingEditor.vue
-> src/components/centralized/project/BuildingEditorDrawer.vue

src/views/centralized/roomType/RoomTypeEditor.vue
-> src/components/centralized/project/RoomTypeEditorDrawer.vue

src/views/centralized/room/CentralizedRoomEditor.vue
-> src/components/centralized/project/CentralizedRoomEditorDrawer.vue

src/views/decentralized/room/DecentralizedRoomEditor.vue
-> src/components/decentralized/community/DecentralizedRoomEditorDrawer.vue
```

### 4. utils 清理 `views/common`

```text
src/views/common/formHelpers.ts
-> 拆分为：
   src/utils/format/display.ts
   src/utils/format/money.ts
   src/utils/format/time.ts
   src/utils/route/query.ts
   src/utils/normalize/taggedImage.ts
   src/utils/normalize/value.ts

src/views/common/listFilterStorage.ts
-> src/utils/storage/listFilterStorage.ts
```

### 5. 删除不再需要的 `views/common`

```text
src/views/common/ModulePlaceholder.vue
-> 删除

src/views/common/
-> 清空后删除目录
```

说明：

- `ModulePlaceholder.vue` 只在确认无引用后删除。
- 本轮不保留新的 `views/common` 替代目录。

### 6. api 重组

```text
src/api/publishAuth.ts
-> src/api/auth/publishAuth.ts

src/api/building.ts
-> src/api/centralized/building.ts

src/api/centralizedProject.ts
-> src/api/centralized/project.ts

src/api/centralizedRoom.ts
-> src/api/centralized/room.ts

src/api/roomType.ts
-> src/api/centralized/roomType.ts

src/api/decentralizedCommunity.ts
-> src/api/decentralized/community.ts

src/api/decentralizedRoom.ts
-> src/api/decentralized/room.ts
```

### 7. model 重组

```text
src/model/publishAuth.ts
-> src/model/auth/publishAuth.ts

src/model/building.ts
-> src/model/centralized/building.ts

src/model/centralizedProject.ts
-> src/model/centralized/project.ts

src/model/centralizedRoom.ts
-> src/model/centralized/room.ts

src/model/roomType.ts
-> src/model/centralized/roomType.ts

src/model/decentralizedCommunity.ts
-> src/model/decentralized/community.ts

src/model/decentralizedRoom.ts
-> src/model/decentralized/room.ts

src/model/common.ts
-> src/model/shared/common.ts

src/model/enums.ts
-> src/model/shared/enums.ts
```

### 8. utils 基础设施重组

```text
src/utils/authToken.ts
-> src/utils/auth/authToken.ts

src/utils/http.ts
-> src/utils/http/http.ts

src/utils/request.ts
-> src/utils/http/request.ts
```

### 9. 样式文件重组

当前：

```text
src/assets/css/common/common.scss
src/assets/css/common/publish-workbench.scss
```

目标拆分为：

```text
src/assets/css/index.scss

src/assets/css/layout/app-layout.scss
src/assets/css/layout/app-header.scss
src/assets/css/layout/app-sidebar.scss

src/assets/css/auth/login-page.scss

src/assets/css/home/home-page.scss

src/assets/css/centralized/project-list-page.scss
src/assets/css/centralized/project-form-page.scss
src/assets/css/centralized/project-detail-page.scss

src/assets/css/decentralized/community-list-page.scss
src/assets/css/decentralized/community-form-page.scss
src/assets/css/decentralized/community-detail-page.scss

src/assets/css/shared/tagged-image-fields.scss
```

说明：

- 不再保留 `publish-workbench.scss` 这类大聚合文件。
- 不把 `scss` 和 `vue` 放在同目录。
- `index.scss` 只负责统一引入，不承载大段具体页面样式。
- 页面级 scss 是默认方案，panel、drawer、普通业务组件的样式优先并入所属页面 scss。
- 只有真正复用且复杂的组件，才单独建立组件级 scss 文件。

## 分阶段执行顺序

### 第一阶段：只出清单文档

目标：

- 固定重构边界。
- 固定目录命名。
- 固定 rename / move 顺序。

产出：

- 本文档。

### 第二阶段：页面和组件目录迁移

执行内容：

1. 页面改名为 `*Page.vue`。
2. `DevTokenDialog.vue`、`TaggedImageFields.vue` 移入 `components/`。
3. 四个 editor 移入 `components/`。
4. 四个 panel 移入 `components/`。
5. 更新所有 import 引用。

完成标准：

- `views/` 里只剩页面和布局壳。
- `components/` 承接业务块和可复用块。

### 第三阶段：清理 `views/common`

执行内容：

1. 拆 `formHelpers.ts` 到 `utils/format`、`utils/route`、`utils/normalize`。
2. 迁移 `listFilterStorage.ts` 到 `utils/storage`。
3. 确认 `ModulePlaceholder.vue` 无引用后删除。
4. 删除 `views/common` 目录。

完成标准：

- `views/common` 不再存在。

### 第四阶段：api / model / utils 二级目录化

执行内容：

1. `api` 按 `auth / centralized / decentralized` 重组。
2. `model` 按 `auth / centralized / decentralized / shared` 重组。
3. `utils` 按 `auth / http / format / route / normalize / storage` 重组。
4. 更新所有 import 路径。

完成标准：

- `api`、`model`、`utils` 不再平铺。

### 第五阶段：样式文件重组

执行内容：

1. 拆 `common.scss` 与 `publish-workbench.scss`。
2. 先按页面建立页面级 scss 文件。
3. layout 拆成独立 `layout/*.scss`。
4. 仅为真正复用且复杂的组件建立单独 scss。
5. 用 `index.scss` 统一引入。
6. 删除旧的聚合样式文件。

完成标准：

- `scss` 全部位于 `src/assets/css/*`。
- 没有内嵌样式。
- 没有 `publish-workbench.scss` 这种大杂烩文件。
- 没有默认给每个 panel、drawer 强拆独立 scss。

### 第六阶段：回归验证

执行内容：

1. 检查所有 import 是否已更新。
2. 检查是否仍有 `views/common` 引用残留。
3. 检查是否仍有旧文件名引用残留。
4. 执行前端本地验证。

建议检查：

```text
rg "views/common|ProjectList.vue|ProjectForm.vue|ProjectDetail.vue|CommunityList.vue|CommunityForm.vue|CommunityDetail.vue|BuildingEditor.vue|RoomTypeEditor.vue|CentralizedRoomEditor.vue|DecentralizedRoomEditor.vue" src
rg "publish-workbench.scss|common/common.scss" src
```

## 实施约束

1. 只改前端代码。
2. 不回滚其他 agent 已有改动。
3. 每一阶段先完成 rename / move，再做路径修正，不混在一起大改。
4. 不顺手做业务逻辑重构。
5. 不顺手做样式视觉改版。
6. 不新增兼容性目录。

## 本清单对应的第一阶段输出

本阶段只新增本文档：

```text
shared_docs/workstreams/publish-web/active/src-structure-refactor.md
```

本阶段不继续改前端代码，不做目录迁移，不做样式迁移。
