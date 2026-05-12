# 聊天与 AI 模块

## 1. 模块目标

本模块用于管理聊天会话、聊天消息、AI 意图识别、需求抽取、推荐记录与客服分配记录。

该模块的核心目标不是单纯保存消息，而是为业务链路提供可回溯、可分析、可触发后续动作的数据基础。

## 2. 集合清单

- `chat_session`
- `chat_message`
- `ai_intent_record`
- `ai_requirement_extract`
- `ai_recommendation_record`
- `ai_assignment_record`

## 3. 集合设计

### 3.1 `chat_session`

**功能说明**  
聊天会话主档。用于描述用户与 AI 或客服之间的一次会话。

**核心字段**

- `session_id`：会话唯一标识
- `user_id`：关联用户标识
- `session_type`：会话类型，例如 AI 对话、人工客服
- `session_status`：会话状态
- `last_message_at`：最后消息时间
- `context_version`：上下文版本
- `created_at`：创建时间
- `updated_at`：更新时间

### 3.2 `chat_message`

**功能说明**  
聊天消息记录。用于保存消息内容、角色、结构化附件与消息顺序。

**核心字段**

- `message_id`：消息唯一标识
- `session_id`：关联 `chat_session.session_id`
- `sender_type`：发送方类型，例如用户、AI、客服
- `message_type`：消息类型，例如文本、推荐卡片、系统提示
- `content`：消息正文
- `attachments`：附加结构化内容
- `created_at`：创建时间

### 3.3 `ai_intent_record`

**功能说明**  
AI 意图识别记录。用于记录会话中识别出的业务意图，例如委托出租、私人定制、预约看房。

**核心字段**

- `intent_record_id`：意图记录唯一标识
- `session_id`：关联会话标识
- `message_id`：触发消息标识
- `intent_code`：意图编码
- `confidence_score`：置信度
- `trigger_text`：触发文本
- `created_at`：创建时间

### 3.4 `ai_requirement_extract`

**功能说明**  
AI 需求抽取记录。用于保存 AI 从对话中抽取出的找房条件结构。

**核心字段**

- `extract_id`：抽取记录唯一标识
- `session_id`：关联会话标识
- `message_id`：来源消息标识
- `requirement_snapshot`：结构化需求快照
- `extract_status`：抽取状态
- `created_at`：创建时间

### 3.5 `ai_recommendation_record`

**功能说明**  
AI 推荐记录。用于记录某次会话中 AI 推荐了哪些房源，以及推荐依据。

**核心字段**

- `recommendation_id`：推荐记录唯一标识
- `session_id`：关联会话标识
- `listing_ids`：推荐房源标识集合
- `reason_summary`：推荐说明
- `created_at`：创建时间

### 3.6 `ai_assignment_record`

**功能说明**  
AI 分配记录。用于记录用户需求被分配给哪位员工、采用了何种分配策略。

**核心字段**

- `assignment_id`：分配记录唯一标识
- `session_id`：关联会话标识
- `customer_id`：关联客户标识
- `assigned_staff_id`：被分配员工
- `assignment_rule`：分配规则说明
- `assignment_reason`：分配原因
- `created_at`：创建时间
