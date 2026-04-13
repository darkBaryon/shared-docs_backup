# AI 找房系统数据库设计文档

## 1. 总体设计思路

- **User (用户持久层)**：记录用户的长期静态属性，跨会话共享。
- **Session (会话逻辑层)**：记录单次对话的业务进度，由状态机驱动。
- **Message (流水层)**：记录原始聊天记录，用于上下文追溯。
- **Redis 缓存层**：活跃会话仅在 Redis 中读写，满足结束条件后才批量落库 MongoDB。

---

## 2. MongoDB 数据结构定义 (Collections)

### 2.1 `hs_usr_user` (用户档案表)

**职责**：全局唯一用户表，同时服务于会话模块与房源模块。

| 字段名                  | 类型     | 必填 | 描述                                                  |
| -------------------- | ------ | -- | --------------------------------------------------- |
| `user_id`            | String | 是  | 业务唯一标识（建立唯一索引）                                      |
| `role`               | String | 否  | 角色控制：`landlord`（房东）/ `tenant`（租客），默认 `tenant`        |
| `nickname`           | String | 否  | 用户显示名称                                              |
| `contact`            | Object | 否  | 联系方式：`{"wechat": "string", "phone": "string"}`      |
| `global_preferences` | Object | 是  | 全局找房偏好（如：只要个人房源、有宠物），默认 `{}`                        |
| `status`             | Int    | 是  | 状态：`1` 正常，写入时固定为 `1`                                |
| `created_at`         | Int64  | 是  | 注册时间（Unix 秒级时间戳，`$setOnInsert` 写入，不会被后续 upsert 覆盖） |
| `last_active`        | Int64  | 是  | 最后一次进入系统的时间（Unix 秒级时间戳，每次请求 `$set` 更新）              |

**索引**：`{"user_id": 1, unique: true}`

---

### 2.2 `hs_chat_session` (会话状态表)

**职责**：存储单次任务的动态状态（任务隔离核心）。落库时机：ENDING 触发或超时定时刷盘。

| 字段名               | 类型    | 必填 | 描述                                                            |
| ----------------- | ----- | -- | ------------------------------------------------------------- |
| `_id`             | String | 是  | MongoDB 原生主键，值为预生成的 ObjectId 字符串（`str(ObjectId())`），在 Redis 写入前生成后沿用至落库 |
| `user_id`         | String | 是  | 所属用户 ID（建立索引）                                                |
| `state`           | Int   | 是  | 当前状态机位置（枚举整数值，如 `GREETING=1`, `COLLECTING=2`, `SEARCHING=3`） |
| `requirement`     | Object | 否  | 本次会话提取的需求快照（区域、预算、户型），默认 `{}`                               |
| `off_topic_count` | Int   | 是  | 离题计数器，用于引导回正题，默认 `0`                                        |
| `status`          | Int   | 是  | 状态：`1` 正常，写入时固定为 `1`                                        |
| `created_at`      | Int64 | 是  | 会话开启时间（Unix 秒级时间戳）                                           |
| `updated_at`      | Int64 | 是  | 最后一次交互时间（Unix 秒级时间戳，用于超时判定）                                 |

**索引**：`{"user_id": 1}`（`_id` 为 MongoDB 原生主键，自带唯一性，无需额外建索引）

---

### 2.3 `hs_chat_message` (聊天流水表)

**职责**：存储特定会话的原始对话流，落库时先清空旧数据再批量插入（`delete_many` + `insert_many`）。

| 字段名          | 类型    | 必填 | 描述                                   |
| ------------ | ----- | -- | ------------------------------------ |
| `session_id` | String | 是  | 关联的会话 ID（建立索引）                       |
| `role`       | String | 是  | 角色枚举：`user`（用户）/ `assistant`（AI）     |
| `content`    | String | 是  | 消息文本内容                               |
| `status`     | Int   | 是  | 状态：`1` 正常，写入时固定为 `1`                |
| `timestamp`  | Int64 | 是  | 消息产生时间（Unix 秒级时间戳）                   |

**索引**：`{"session_id": 1}`

---

## 3. Redis 缓存结构

活跃会话期间所有读写均在 Redis 完成，键前缀统一为 `hs:chat:sess:info`，TTL 均为 86400 秒（24h）。

| 键模式                                   | 类型   | 描述                                          |
| ------------------------------------- | ---- | ------------------------------------------- |
| `hs:chat:sess:info:active:{user_id}`  | String | 存储该用户当前活跃的 `session_id`                    |
| `hs:chat:sess:info:meta:{session_id}` | String | JSON 序列化的会话元数据（对应 `hs_chat_session` 字段结构）  |
| `hs:chat:sess:info:msg:{session_id}`  | List | 消息流水（每条为 JSON 字符串），最多保留 500 条（`LTRIM` 截断）   |
| `hs:chat:sess:info:heartbeat`         | ZSet | 以 Unix 时间戳为 score，member 为 `session_id`，用于超时扫描 |
| `hs:lock:flush_stale`                 | String | 分布式锁，TTL 60s，防止多节点并发刷盘                     |

---

## 4. 会话生命周期管理 (Lifecycle)

### 4.1 会话结束 (ENDING) 判定

系统通过 `SessionManager` 实现 **Write-Back（回写）** 策略，在以下任一条件达成时强制落库 MongoDB 并销毁 Redis 缓存：

1. **状态机判定（显式结束）**：`DialogueOrchestrator` 返回 `DialogueState.ENDING` 时，`update_chat` 立即触发落库。
2. **超时自动判定（隐式结束）**：`now - updated_at >= session_timeout_seconds`（默认 1800s）时，下次请求进入或定时任务扫描时触发落库，并生成全新 `session_id` 开启新会话。

### 4.2 性能优化 (I/O 减压)

- **读写分离**：实时对话仅在 Redis 缓存中更新，不触碰 MongoDB。
- **批量入库**：仅在满足上述"结束"条件时执行 MongoDB 写入，消息采用 `insert_many` 批量写入。
- **落库安全**：仅在落库成功后才清理 Redis 数据；若落库失败则保留 Redis 会话，避免数据丢失。
