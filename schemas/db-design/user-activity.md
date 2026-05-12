# 用户行为模块

## 1. 模块目标

本模块用于记录用户在小程序侧产生的行为数据，包括收藏、浏览足迹、看房计划与站内通知。

该模块用于支撑“我的”页、推荐系统、行为分析和运营触达。

## 2. 集合清单

- `usr_favorite`
- `usr_history`
- `usr_plan`
- `usr_notification`

## 3. 集合设计

### 3.1 `usr_favorite`

**功能说明**  
收藏记录。用于记录用户主动收藏的房源。

**核心字段**

- `favorite_id`：收藏记录唯一标识
- `user_id`：关联用户标识
- `listing_id`：关联房源发布单标识
- `created_at`：收藏时间

### 3.2 `usr_history`

**功能说明**  
浏览足迹。用于记录用户查看过的房源详情行为。

**核心字段**

- `history_id`：足迹记录唯一标识
- `user_id`：关联用户标识
- `listing_id`：关联房源发布单标识
- `viewed_at`：浏览时间

### 3.3 `usr_plan`

**功能说明**  
看房计划。用于记录用户已加入计划、待安排或待跟进的房源集合。

**核心字段**

- `plan_id`：计划唯一标识
- `user_id`：关联用户标识
- `listing_ids`：计划中的房源集合
- `plan_status`：计划状态
- `remark`：备注
- `updated_at`：更新时间

### 3.4 `usr_notification`

**功能说明**  
用户通知。用于存储站内通知、预约结果通知、推荐通知等面向用户的消息。

**核心字段**

- `notification_id`：通知唯一标识
- `user_id`：关联用户标识
- `notification_type`：通知类型
- `title`：通知标题
- `content`：通知内容
- `is_read`：是否已读
- `related_id`：关联业务对象标识
- `created_at`：创建时间
