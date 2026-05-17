# 房源管理总览模块需求文档

## 1. 背景

房源管理总览属于 admin MVP Phase 3，服务对象是平台运营员工。

第一阶段 house 模块只做平台视角只读查询，不做后台直接编辑房源服务信息、联系人信息、上下架动作或审核动作。

后台房源列表主读：

- `hs_hpd_admin_listing`

详情第一阶段也以 `hs_hpd_admin_listing` 为摘要基础；如页面需要完整 HMD 长字段，再按 `source_type/source_id` 补充详情接口字段。

## 2. 目标

第一阶段目标：

1. 有权限员工可以先查看项目/小区列表。
2. 有权限员工可以在项目/小区下继续查看楼栋或房间列表。
3. 有权限员工可以查看单个房间详情摘要。
4. 支持按发房方、资产类型、城市区域、房态、发布状态、审核状态筛选。
5. 后台查看不改变房源状态。

## 3. 非目标

第一阶段不做：

- 后台编辑房源基础信息。
- 后台编辑服务信息或联系人信息。
- 上架、下架、恢复、标记出租等发布动作。
- 审核通过 / 驳回。
- 导出文件真实生成。
- 复杂全文搜索或跨集合聚合查询。

## 4. 用户故事

### 4.1 查看项目/小区列表

作为运营员工，我需要先查看平台项目/小区入口，以便再逐级进入楼栋和房间。

验收：

- 可以分页查看。
- 可以按发房方筛选。
- 可以按资产类型筛选。
- 可以按城市、区域筛选。
- 可以按房态、发布状态、审核状态筛选。
- 列表展示项目/小区名称、发房方、城市区域、资产类型、楼栋数、房间数、更新时间。
- 无 `house.view` 权限时不能查看。

### 4.2 查看楼栋/房间列表

作为运营员工，我需要在选中项目/小区后逐级查看楼栋和房间，以便定位具体房源。

验收：

- 集中式项目可以继续查看楼栋列表。
- 分散式小区可以直接进入房间列表。
- 房间列表展示房源标题、发房方、城市区域、项目/小区、楼栋、租金、房态、发布状态、审核状态、更新时间。
- 无 `house.view` 权限时不能查看。

### 4.3 查看房间详情

作为运营员工，我需要查看单个房间当前状态，以便判断是否可展示、是否待审核、归属哪个发房方。

验收：

- `listing_id` 必填。
- 可以看到发房方、位置、房源摘要、房态、发布状态、审核状态。
- 房源不存在或已删除时返回明确中文提示。
- 查看详情不改变任何房源状态。

## 5. 业务规则

### 5.1 主键和命名

- 接口层统一使用 `listing_id`。
- `listing_id` 等于 `hs_hpd_listing._id`。
- 后台列表项来自 `hs_hpd_admin_listing`。

### 5.2 数据边界

- `hs_hpd_admin_listing` 是 read model，不是业务主表。
- 后台查询不直接聚合 `hs_hmd_*`、`hs_hpd_listing`、`hs_hac_*`。
- HMD、HPD、HAC 变化后由 projection 或业务 service 刷新 `hs_hpd_admin_listing`。

### 5.3 权限

- `house_root/list`、`house_building/list`、`house_room/list`、`house_room/detail` 需要 `house.view`。
- 第一阶段没有 `house.edit` 接口。
- `house_room/export` 只保留接口名，不进入实现。

## 6. 页面字段

第一阶段列表字段：

- `listing_id`
- `source_type`
- `source_id`
- `asset_mode`
- `provider_id`
- `provider_phone`
- `provider_name`
- `root_type`
- `root_id`
- `project_id`
- `project_name`
- `building_id`
- `building_name`
- `room_type_id`
- `room_type_name`
- `decentralized_id`
- `community_name`
- `rent_mode`
- `city`
- `district`
- `biz_area`
- `room_no`
- `title`
- `price`
- `price_text`
- `layout_text`
- `area_size`
- `room_status`
- `listing_status`
- `audit_status`
- `is_online`
- `updated_at`

## 7. 接口范围

第一阶段实现：

- `POST /api/v1/house_root/list`
- `POST /api/v1/house_building/list`
- `POST /api/v1/house_room/list`
- `POST /api/v1/house_room/detail`

第一阶段只保留文档，不实现：

- `POST /api/v1/house_room/export`

## 8. 任务拆分

文档：

- 补 house 需求文档。
- 补 house 一接口一文档。
- 更新 admin API 和实施计划。

后端：

- 补 `hs_hpd_admin_listing` schema、model、repository。
- 接入 admin listing projection。
- 新增 `service/admin/house`。
- 更新 `handler/v1/admin/house` 为分层查询路由。
- 接入 wire 和 admin protected routes。

测试：

- projection 单测覆盖 admin read model 映射和 lifecycle refresh。
- service 单测覆盖列表、详情、筛选校验、not found。
- handler 单测覆盖成功响应、权限拒绝、JSON 错误和 service 错误透传。
