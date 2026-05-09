# 发房系统 API 接口文档

## 1. 适用范围

- 本文档用于定义发房系统接口命名与第一期接口范围。
- 发房系统模块统一使用 `publish` 作为 API 模块名。
- 当前第一期 HMD 录入与维护后端链路已通过真实服务联调，但发房系统整体语义不局限于 HMD，后续可继续扩展到 HPD 发布与展示层。

## 2. 路径命名规则

路径命名遵守 [../overview/project-spec.md](../overview/project-spec.md)。

统一命名格式：

```text
POST /api/v{version}/{module}/{action}
```

发房系统固定为：

```text
POST /api/v1/publish/{action}
```

约定：

- `{module}` 固定为 `publish`
- `{action}` 使用业务动作命名，不使用 REST 风格 `GET/PUT/DELETE`
- `{action}` 必须明确对象与动作，避免仅使用 `create`、`update`、`detail` 这类无对象前缀的名字

示例：

- `create_centralized_project`
- `list_buildings_by_project`
- `update_decentralized_room_status`

## 3. 业务边界

- 发房系统接口是发布业务入口，不是单纯的 HMD 表 CRUD 直通接口。
- 发房系统 service 负责业务编排，可以同时协调：
  - HMD 主数据写入
  - HPD 发布层数据生成/刷新
  - 房东端展示数据
  - 小程序用户展示数据
  - 后台展示数据
- repository 层仍按数据域拆分，例如 `repository/hmd`、后续 `repository/hpd`。

## 4. 第一期开口范围

第一期先实现发房系统的 HMD 录入、详情、列表、更新、房态维护。

### 4.1 集中式项目

- `POST /api/v1/publish/create_centralized_project`
- `POST /api/v1/publish/centralized_project_detail`
- `POST /api/v1/publish/update_centralized_project`
- `POST /api/v1/publish/list_centralized_projects`

### 4.2 集中式楼栋

- `POST /api/v1/publish/create_building`
- `POST /api/v1/publish/building_detail`
- `POST /api/v1/publish/update_building`
- `POST /api/v1/publish/list_buildings_by_project`

### 4.3 集中式房型模板

- `POST /api/v1/publish/create_room_type`
- `POST /api/v1/publish/room_type_detail`
- `POST /api/v1/publish/update_room_type`
- `POST /api/v1/publish/list_room_types_by_project`
- `POST /api/v1/publish/list_room_types_by_building`

### 4.4 集中式房间

- `POST /api/v1/publish/create_centralized_room`
- `POST /api/v1/publish/centralized_room_detail`
- `POST /api/v1/publish/update_centralized_room`
- `POST /api/v1/publish/update_centralized_room_status`
- `POST /api/v1/publish/list_centralized_rooms_by_project`
- `POST /api/v1/publish/list_centralized_rooms_by_building`

### 4.5 分散式主档（小区）

- `POST /api/v1/publish/create_decentralized_community`
- `POST /api/v1/publish/decentralized_community_detail`
- `POST /api/v1/publish/update_decentralized_community`
- `POST /api/v1/publish/list_decentralized_communities`
- `POST /api/v1/publish/list_decentralized_communities_by_city`
- `POST /api/v1/publish/list_decentralized_communities_by_district`

### 4.6 分散式房间

- `POST /api/v1/publish/create_decentralized_room`
- `POST /api/v1/publish/decentralized_room_detail`
- `POST /api/v1/publish/update_decentralized_room`
- `POST /api/v1/publish/update_decentralized_room_status`
- `POST /api/v1/publish/list_decentralized_rooms_by_community`

## 5. 接口语义约定

### 5.1 `create_*`

- 语义是“在发房系统中创建一个业务对象”
- 不限定为单表插入
- service 可以在内部同时处理 HMD、HPD 及其他衍生数据

### 5.2 `*_detail`

- 按对象 ID 返回详情
- 用于编辑页回填

### 5.3 `update_*`

- 更新基础信息
- 具体可更新字段以 service 与 repository 白名单约束为准

### 5.4 `list_*`

- 用于选择器、列表页、级联选择
- 默认返回轻量列表，不要求等同后台管理系统完整详情结构

### 5.5 `update_*_status`

- 仅更新状态字段
- 不混入基础信息更新

## 6. 通用约定

- 接口统一使用 `POST`
- 除登录/注册等明确公开接口外，默认使用 `Authorization: Bearer <token>`
- 响应结构统一：

```json
{
  "code": 0,
  "error": "",
  "data": {}
}
```

- 业务成功判定：`code === 0`

## 7. 后续扩展

后续发房系统扩展到 HPD 时，仍继续沿用：

```text
POST /api/v1/publish/{action}
```

例如未来可新增：

- `create_listing`
- `update_listing_publish_content`
- `submit_listing_for_audit`
- `offline_listing`

具体是否拆出第二阶段接口，以业务实现进度为准。
