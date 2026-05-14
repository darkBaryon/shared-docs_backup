# 项目通用代码规范

本文档从 `/Users/xinyue/AI找房项目项目规范.docx` 整理而来，是全项目通用规范。语言或端侧的细化规范可以在各自目录补充，但不能和本文档冲突。

## 1. 基本原则

- 约定大于实现习惯，特殊处理必须先确认并补充到共享文档。
- 所有类型默认值都视为“未指定”：
  - `int` 的 `0`
  - `float` 的 `0.0`
  - `string` 的 `""`
  - `pointer` 的 `nil/null`
- 禁止把默认值设计成有意义的业务状态。
- 状态枚举应从正数开始表达有效业务状态，`0` 只表示未设置。

## 2. MongoDB 规范

### 2.1 集合命名

```text
hs_{模块简称}_{业务实体}
```

示例：

- `hs_usr_user`
- `hs_agt_agent`
- `hs_hmd_room_centralized`

### 2.2 字段命名

- 使用小写加下划线。
- 关联字段使用 `{实体}_id`，例如 `user_id`、`house_id`。
- 时间字段使用 `created_at`、`updated_at`。
- `_id` 必须使用 MongoDB `ObjectID`，禁止自定义字符串 ID。
- 外键直接保存关联表 `ObjectID`。

### 2.3 索引命名

单字段索引：

```text
{字段名}_{排序}
```

联合索引：

```text
{字段名}_{排序}_{字段名}_{排序}
```

示例：

- `{ "openid": 1 }` -> `openid_1`
- `{ "status": 1, "created_at": -1 }` -> `status_1_created_at_-1`

### 2.4 状态字段

- 使用整数类型，禁止字符串枚举。
- `0`：未设置 / 初始态。
- `1-99`：有效业务状态。
- `-1 ~ -99`：异常、删除等状态。

### 2.5 设计原则

- 面向查询设计，不按关系型数据库思路建模。
- 能不关联尽量不关联，合理内嵌。
- 禁止无限增长内嵌数组。
- 独立实体分集合，附属数据可内嵌。
- 查询必须走索引，单集合索引数量原则上不超过 5 个。
- 复合索引等值条件在前，范围 / 排序条件在后。
- 原则上禁止物理删除，业务删除使用软删除。
- 禁止多层嵌套超过 3 层。
- 禁止随意增加动态字段；结构变更必须同步 schema 和变更记录。

## 3. Redis 规范

Key 命名格式：

```text
hs:{服务}:{模块}:{业务}:{标识}
```

示例：

- `hs:bs:usr:info:{user_id}`
- `hs:bs:hs:info:{house_id}`
- `hs:chat:sess:state:{session_id}`

使用原则：

- Redis 只用于临时缓存或临时状态。
- 业务数据必须持久化到 MongoDB。
- 所有 key 必须设置 TTL。

## 4. API 规范

### 4.1 路由格式

统一格式：

```text
POST /api/v{version}/{模块}/{动作}
```

示例：

- `POST /api/v1/user/create`
- `POST /api/v1/user/list`
- `POST /api/v1/house/search`
- `POST /api/v1/appointment/create`

### 4.2 命名要求

- 所有业务接口统一使用 `POST`。
- 禁止把主业务接口设计成 RESTful 风格的 `GET/PUT/DELETE`。
- `{模块}` 是业务模块名，例如 `auth`、`user`、`house`。发房系统按业务对象拆 module，例如 `centralized_project`、`building`、`room_type`、`centralized_room`、`decentralized_community`、`decentralized_room`。
- `{动作}` 必须是单个 path segment，可以用 snake_case 表达具体动作。
- 禁止在 `{动作}` 后继续追加业务层级。

正确：

```text
POST /api/v1/house/search
POST /api/v1/house/public_detail
POST /api/v1/centralized_project/create
POST /api/v1/decentralized_room/update_status
```

错误：

```text
POST /api/v1/miniapp/listing/search
POST /api/v1/publish/create_centralized_project
POST /api/v1/publish/hmd/{domain}/{action}
```

说明：

- 小程序是前端端类型，不天然等于 API 模块。
- 小程序找房接口当前归属 `house` 模块。
- 发房系统是端侧/业务域名称，API module 位使用具体业务对象，不使用 `publish`。

### 4.3 响应格式

原始项目规范使用：

```json
{
  "code": 0,
  "message": "success",
  "data": {}
}
```

当前 Go 后端已统一为：

```json
{
  "code": 0,
  "error": "",
  "data": {}
}
```

Go 后端接口以 [../api/error-codes.md](../api/error-codes.md) 和各模块 API 文档为准。

## 5. 结构体命名规范

通用规则：首字母大写驼峰。

建议后缀：

| 类型 | 后缀 | 示例 |
| --- | --- | --- |
| 数据库映射结构体 | `Db` | `UserDb`, `HouseDb` |
| 请求入口结构体 | `Req` | `HouseCreateReq`, `HouseSearchReq` |
| 响应结构体 | `Resp` | `HouseResp`, `UserResp` |
| 列表响应 | `ListResp` | `HouseListResp` |

Go 后端当前已经存在的 domain model 不因为本文档做批量重命名；新增 HTTP 请求 / 响应 DTO 时优先遵循 `Req`、`Resp`、`ListResp` 语义。
