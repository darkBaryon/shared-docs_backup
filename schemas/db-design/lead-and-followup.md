# 客户与业务跟进模块

## 1. 模块目标

本模块用于描述客户、找房需求、预约看房、跟进过程以及推荐匹配关系。

该模块是找房业务闭环的核心，不应仅依赖聊天记录或收藏记录来表达客户状态。

## 2. 集合清单

- `lead_customer`
- `lead_requirement`
- `appointment_viewing`
- `lead_followup`
- `lead_match`

## 3. 集合设计

### 3.1 `lead_customer`

**功能说明**  
客户主档。用于表达业务上的客户对象，不等同于单纯的登录用户身份。

**核心字段**

- `customer_id`：客户唯一标识
- `user_id`：关联小程序用户标识
- `name`：客户姓名或称呼
- `phone`：手机号
- `source_channel`：来源渠道
- `customer_tags`：客户标签
- `owner_staff_id`：当前归属员工
- `status`：客户状态
- `created_at`：创建时间
- `updated_at`：更新时间

### 3.2 `lead_requirement`

**功能说明**  
找房需求记录。用于描述客户某一次明确的找房意图和条件快照。

**核心字段**

- `requirement_id`：需求唯一标识
- `customer_id`：关联 `lead_customer.customer_id`
- `budget_min`：预算下限
- `budget_max`：预算上限
- `preferred_areas`：偏好区域
- `rent_mode`：租住方式
- `move_in_plan`：入住计划
- `requirement_status`：需求状态
- `source_type`：需求来源，例如主动填写、AI 抽取、客服录入
- `created_at`：创建时间
- `updated_at`：更新时间

### 3.3 `appointment_viewing`

**功能说明**  
预约看房记录。用于描述用户预约某套房源、预约时段及后续状态。

**核心字段**

- `appointment_id`：预约唯一标识
- `customer_id`：关联客户标识
- `listing_id`：关联房源发布单标识
- `appointment_time`：预约时间
- `appointment_status`：预约状态
- `assigned_staff_id`：服务员工
- `remark`：预约备注
- `created_at`：创建时间
- `updated_at`：更新时间

### 3.4 `lead_followup`

**功能说明**  
客户跟进记录。用于沉淀客服和运营在跟进过程中的动作与结论。

**核心字段**

- `followup_id`：跟进唯一标识
- `customer_id`：关联客户标识
- `requirement_id`：关联需求标识
- `staff_id`：跟进员工
- `followup_type`：跟进类型，例如电话、IM、预约确认、带看反馈
- `content`：跟进内容
- `next_action`：下一步动作
- `created_at`：创建时间

### 3.5 `lead_match`

**功能说明**  
推荐匹配记录。用于记录某次客户需求匹配到的房源集合及匹配结果。

**核心字段**

- `match_id`：匹配记录唯一标识
- `customer_id`：关联客户标识
- `requirement_id`：关联需求标识
- `listing_ids`：命中的房源发布单集合
- `match_reason`：匹配理由
- `created_by`：生成方式，例如 AI、客服、系统规则
- `created_at`：创建时间
