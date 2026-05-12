# 运营与营销模块

## 1. 模块目标

本模块用于管理前台可配置的运营位、广告位、弹窗与消息推送任务。

该模块面向后台运营能力建设，不应与房源主数据或聊天记录直接混存。

## 2. 集合清单

- `ops_banner`
- `ops_popup`
- `ops_featured_slot`
- `ops_push_task`

## 3. 集合设计

### 3.1 `ops_banner`

**功能说明**  
Banner 配置。用于管理首页、我的页等位置的轮播图与运营入口。

**核心字段**

- `banner_id`：Banner 唯一标识
- `banner_name`：Banner 名称
- `position_code`：投放位置编码
- `image_url`：图片地址
- `target_type`：跳转目标类型
- `target_value`：跳转目标值
- `sort_order`：排序值
- `status`：状态
- `effective_from`：生效时间
- `effective_to`：失效时间

### 3.2 `ops_popup`

**功能说明**  
弹窗配置。用于管理首页或特定场景下的弹窗活动与公告。

**核心字段**

- `popup_id`：弹窗唯一标识
- `popup_name`：弹窗名称
- `image_url`：弹窗图片
- `content`：弹窗内容
- `target_type`：跳转目标类型
- `target_value`：跳转目标值
- `trigger_scene`：触发场景
- `status`：状态
- `effective_from`：生效时间
- `effective_to`：失效时间

### 3.3 `ops_featured_slot`

**功能说明**  
运营推荐位。用于管理优质公寓、置顶房源、专题入口等运营资源位。

**核心字段**

- `slot_id`：推荐位唯一标识
- `slot_name`：推荐位名称
- `slot_type`：推荐位类型
- `listing_ids`：绑定的房源集合
- `city`：投放城市
- `sort_order`：排序值
- `status`：状态
- `effective_from`：生效时间
- `effective_to`：失效时间

### 3.4 `ops_push_task`

**功能说明**  
推送任务。用于管理系统通知、营销消息、业务提醒等推送动作。

**核心字段**

- `push_task_id`：推送任务唯一标识
- `task_type`：任务类型
- `target_scope`：目标范围
- `target_ids`：目标对象集合
- `content`：推送内容
- `delivery_channel`：发送渠道
- `task_status`：任务状态
- `scheduled_at`：计划发送时间
- `created_at`：创建时间
