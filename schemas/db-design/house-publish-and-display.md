# 房源发布与展示模块

## 1. 模块目标

本模块用于描述：

- 哪些房源当前可对外发布
- 前台搜索与详情应读取哪一层数据
- 房源对外联系信息如何确定

该模块是房源主数据与小程序前台之间的桥接层。

## 2. 集合清单

- `house_listing`
- `house_listing_index`
- `house_contact`
- `house_entrust_relation`

## 3. 集合设计

### 3.1 `house_listing`

**功能说明**  
房源发布单。用于表示一个真实房源对象被组织为一个可对外展示、可推荐、可上下架的发布实体。

**核心字段**

- `listing_id`：发布单唯一标识
- `source_type`：发布来源类型，例如房型、房间、分散式整租、分散式合租
- `source_id`：来源对象标识
- `publish_title`：对外展示标题
- `publish_subtitle`：对外展示副标题
- `rent_mode`：租住方式
- `listing_status`：发布状态，例如草稿、待审核、已上架、已下架、已出租
- `cover`：封面图
- `gallery`：房源图片集
- `price`：对外展示价格
- `tags`：对外展示标签
- `contact_id`：当前联系信息标识
- `published_at`：上架时间
- `updated_at`：更新时间

### 3.2 `house_listing_index`

**功能说明**  
房源展示索引。用于承载前台搜索、推荐、地图找房、详情聚合等高频读取场景的冗余检索对象。

**核心字段**

- `listing_id`：关联 `house_listing.listing_id`
- `city`：城市
- `district`：区域
- `biz_area`：商圈
- `community_name`：小区名称
- `geo`：坐标信息
- `title`：前台展示标题
- `subtitle`：前台展示副标题
- `price_text`：前台展示价格文本
- `layout_text`：前台展示户型文本
- `feature_flags`：结构化筛选标记
- `platform_tags`：平台运营标签
- `weight_score`：排序权重
- `is_online`：当前是否可展示
- `updated_at`：更新时间

### 3.3 `house_contact`

**功能说明**  
房源联系信息。用于维护前台详情页和运营侧展示的最终联系结果。

**核心字段**

- `contact_id`：联系信息唯一标识
- `contact_name`：联系人名称
- `contact_phone`：联系电话
- `contact_qr_code`：联系二维码
- `contact_type`：联系人类型，例如发房方、客服、运营
- `staff_id`：关联员工标识
- `status`：联系信息状态
- `updated_at`：更新时间

### 3.4 `house_entrust_relation`

**功能说明**  
房源委托关系。用于描述发房方、委托人、维护人、对接客服之间的业务关系。

**核心字段**

- `relation_id`：委托关系唯一标识
- `listing_id`：关联 `house_listing.listing_id`
- `owner_name`：发房方或业主名称
- `owner_phone`：发房方联系电话
- `maintainer_staff_id`：当前维护员工
- `service_staff_id`：当前服务员工
- `relation_status`：关系状态
- `effective_from`：生效时间
- `effective_to`：失效时间
- `updated_at`：更新时间
