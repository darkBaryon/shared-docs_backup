# 后端业务流日志计划

状态：执行中。

## 目标

把当前“只有 access log、几乎没有业务流日志”的状态，收成一套可读、可排障、可追请求链的日志体系。

目标不是“多打印一点错误”，而是让日志能够表达：

- 一个请求进入了哪个接口；
- 在 service 里走到了哪几个关键步骤；
- 哪一步成功、哪一步被 scope 拒绝、哪一步失败；
- access log 与 business flow log 可以通过同一个 `request_id` 串起来。

## 设计原则

### 1. access log 和 business flow log 分层

- `middleware`：
  - 只负责统一 access log。
  - 成功请求保持短格式，只记录请求摘要。
  - 失败请求补充最小必要诊断字段。
- `service/domain`：
  - 负责 business flow log。
  - 成功和失败都要打，日志要能表达业务过程，而不只是报错。

### 2. 日志优先按接口链路组织

以接口为单位梳理日志，而不是零散地“哪个文件方便就在哪儿打”。

每个接口至少覆盖：

- `start`
- 关键业务步骤
- `success`
- `denied` / `failed`

### 3. request_id 必须贯穿

- middleware 生成或继承 `X-Request-ID`
- 注入 request context
- service/domain 业务日志从 context 读取 `request_id`
- access log 与 business flow log 用同一个 `request_id` 串联

### 4. 成功日志不省略

日志不是错误打印器，而是业务流记录器。

因此：

- create / update / list / detail / login / logout
- 都应在关键节点打成功日志

### 5. 不在 repository 大量铺日志

- repository 只在必要的基础设施异常场景补上下文
- 正常业务流日志优先写在 service

## 分层约定

### access log

位置：

- `internal/middleware/logger.go`

职责：

- 请求入口和出口摘要
- HTTP status、latency、request_id、principal 摘要

格式要求：

- 成功请求保持短
- 不承载完整业务解释

### business flow log

位置：

- `internal/service/publish/*`
- `internal/service/publish/auth/*`
- 后续按需要扩到 `miniapp`、`domain/listingprojection`、`domain/publishaccess`

职责：

- 记录接口对应的业务流步骤
- 打印关键输入摘要、scope 结果、关键对象 ID、返回数量、补偿回滚结果

事件命名建议：

```text
publish.project.create.start
publish.project.create.hmd_created
publish.project.create.projection_applied
publish.project.create.root_scope_created
publish.project.create.success
publish.project.create.rollback_started
publish.project.create.rollback_succeeded
publish.project.create.rollback_failed
```

列表类接口建议：

```text
publish.project.list.start
publish.project.list.scope_resolved
publish.project.list.success
```

权限拒绝建议：

```text
publish.building.list.denied
publish.room.update.denied
```

## 第一批实施范围

按 publish 接口逐个补：

### publish_auth

- `POST /api/v1/publish_auth/login`
- `POST /api/v1/publish_auth/session`
- `POST /api/v1/publish_auth/logout`

### centralized_project

- `create`
- `detail`
- `list`
- `update`

### building

- `create`
- `detail`
- `list_by_project`
- `update`

### room_type

- `create`
- `detail`
- `list_by_project`
- `list_by_building`
- `update`

### centralized_room

- `create`
- `detail`
- `list_by_project`
- `list_by_building`
- `update`
- `update_status`

### decentralized_community

- `create`
- `detail`
- `list`
- `update`

### decentralized_room

- `create`
- `detail`
- `list_by_community`
- `update`
- `update_status`

## 每类接口的日志要求

### create / update

至少包含：

- `start`
- scope 校验通过 / 拒绝
- HMD write 成功 / 失败
- projection apply 成功 / 失败
- root scope create 成功 / 失败（如适用）
- rollback 开始 / 成功 / 失败（如适用）
- `success`

### detail

至少包含：

- `start`
- 对象读取成功 / 未找到
- scope 校验通过 / 拒绝
- `success`

### list

至少包含：

- `start`
- scope 解析结果
- 列表返回数量
- `success`

### auth

至少包含：

- login start / success / failed
- session start / success / failed
- logout start / success / failed

## 验收标准

1. 成功请求的 access log 保持短格式。
2. publish 每个接口都至少有一条来自处理文件的 business flow log。
3. 关键失败场景可从日志里看出：
   - 哪个接口
   - 哪个步骤
   - 哪个对象 ID
   - 是否发生回滚
4. access log 与 business flow log 使用同一个 `request_id`。
