# 房源主数据模块

## 1. 模块目标

本模块用于描述房源的真实业务对象，是后台录入、维护、审核和发布的基础数据层。

本模块不直接面向小程序搜索结果输出，而是作为房源发布与展示模块的数据来源。

## 2. 集合清单

- `house_project`
- `house_building`
- `house_community`
- `house_room_type`
- `house_room`

## 3. 集合设计

### 3.1 `house_project`

**功能说明**  
项目主档。主要服务集中式公寓、品牌公寓等项目化房源管理。

**核心字段**

- `project_id`：项目唯一标识
- `project_name`：项目名称
- `project_code`：项目编码
- `city`：所在城市
- `district`：所在区域
- `address_text`：详细地址
- `geo`：项目坐标
- `brand_name`：所属品牌
- `status`：项目状态
- `created_at`：创建时间
- `updated_at`：更新时间

### 3.2 `house_building`

**功能说明**  
楼栋主档。用于描述项目下的具体楼栋或物业单元。

**核心字段**

- `building_id`：楼栋唯一标识
- `project_id`：关联 `house_project.project_id`
- `building_name`：楼栋名称
- `building_code`：楼栋编码
- `floor_total`：总楼层数
- `status`：楼栋状态
- `created_at`：创建时间
- `updated_at`：更新时间

### 3.3 `house_community`

**功能说明**  
小区主档。主要服务分散式房源管理。

**核心字段**

- `community_id`：小区唯一标识
- `community_name`：小区名称
- `city`：所在城市
- `district`：所在区域
- `biz_area`：所属商圈
- `address_text`：详细地址
- `geo`：小区坐标
- `subway_station`：最近地铁站
- `status`：小区状态
- `created_at`：创建时间
- `updated_at`：更新时间

### 3.4 `house_room_type`

**功能说明**  
房型主档。用于描述一类可重复复用的户型模板。

**核心字段**

- `room_type_id`：房型唯一标识
- `project_id`：关联 `house_project.project_id`
- `building_id`：关联 `house_building.building_id`
- `room_type_name`：房型名称
- `room_count`：室数
- `hall_count`：厅数
- `bathroom_count`：卫数
- `area_size`：面积
- `orientation`：朝向
- `default_facilities`：默认设施配置
- `status`：房型状态
- `created_at`：创建时间
- `updated_at`：更新时间

### 3.5 `house_room`

**功能说明**  
房间主档。用于描述可实际出租、可维护状态、可关联发布的具体房间。

**核心字段**

- `room_id`：房间唯一标识
- `project_id`：关联 `house_project.project_id`
- `building_id`：关联 `house_building.building_id`
- `community_id`：关联 `house_community.community_id`
- `room_type_id`：关联 `house_room_type.room_type_id`
- `room_no`：房间号
- `floor_no`：所在楼层
- `rent_mode`：租住方式，例如整租、合租
- `layout_text`：户型描述
- `status`：房间状态，例如空置、已租、停用
- `created_at`：创建时间
- `updated_at`：更新时间
