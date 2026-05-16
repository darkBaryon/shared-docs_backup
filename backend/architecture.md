# Backend Architecture

## 1. 目标

这份文档用于回答三类问题：

- 一个功能应该放在哪一层。
- 一层代码可以依赖谁，不可以依赖谁。
- 目录和包名为什么这么取，后续新增模块时按什么规则扩展。

当前项目是新项目，不做历史兼容设计。架构判断只服务当前确认需求，不为旧前端、旧接口、旧数据库口径兜底。

## 2. 总体分层

Go 后端采用下面这条主链路：

```text
router
  -> handler/v1/{terminal}/{module}
    -> service/{terminal}/{module}
      -> domain/{capability}
      -> repository/{data-subdomain}
        -> pkg/database
```

其中：

- `terminal` 是前端终端，例如 `miniapp`、`publish`、`admin`。
- `module` 是这个终端下的接口模块，例如 `auth`、`house`、`favorite`、`history`。
- `capability` 是内部领域能力，不直接暴露为某个前端接口模块。

这条链路不是要求每个请求必须经过所有层，而是要求依赖方向始终单向：

- `router -> handler -> service`
- `service -> domain / repository`
- `domain -> repository`
- `repository -> model / pkg/database`

禁止反向依赖：

- `repository` 不能依赖 `service`、`handler`
- `domain` 不能依赖 `handler`
- `model` 不能依赖 `repository`、`service`

## 3. 分层职责

### 3.1 router

职责：

- 注册 HTTP 路由。
- 组合 middleware。
- 组织不同 terminal 的路由分组。

应该放什么：

- `POST /api/v1/...` 路径注册
- 认证 middleware 挂载
- 终端级 route group 组织

不应该放什么：

- DTO 解析
- 业务判断
- repository 调用

命名依据：

- 只按 HTTP 路由结构组织，不承载业务命名判断。

### 3.2 handler

职责：

- 接收 HTTP 请求。
- 读取 path/query/body/header。
- 完成传输层 DTO 校验和基础类型转换。
- 调用对应 terminal 的 service。
- 把 service 返回结果映射成统一响应结构。

应该放什么：

- `BindJSON`、query/path 参数解析
- `ObjectID` 字符串转 `bson.ObjectID`
- 必填字段、字段格式、分页参数的基础校验
- `response.Success` / `response.Err`

不应该放什么：

- 业务规则判断
- 权限判定细节
- 跨 collection 一致性校验
- repository 直接调用

边界说明：

- handler 只知道“这是哪个终端、哪个接口模块、这个接口要什么入参和出参”。
- handler 不应该知道 Mongo 字段白名单、投影规则、状态流转细节。

命名依据：

- 一律按 `handler/v1/{terminal}/{module}` 拆分。
- 一个 handler 包只服务一个 terminal 下的一个接口模块。

### 3.3 service

职责：

- 表达某个终端、某个接口模块的应用服务入口。
- 编排当前接口需要的业务步骤。
- 负责业务规则、权限、数据作用域、状态流转。
- 决定要不要调用 domain，或者直接调用 repository。

应该放什么：

- “小程序登录成功后发 session token”
- “publish 登录后组装 principal、角色、权限”
- “详情页按登录态决定是否返回收藏状态”
- “列表查询先归一化筛选条件，再调用 read model repository”

不应该放什么：

- HTTP DTO 绑定细节
- Mongo update doc 拼装细节
- 通用 collection CRUD

边界说明：

- service 是前端视角的入口层，和某个终端强绑定。
- 同样是 `auth`，`miniapp/auth` 和 `publish/auth` 是两个不同 service 边界，不能为了名字相同强行复用。
- 如果某段逻辑只服务一个 terminal，就优先留在对应 service 内部，不要过早上提为 domain。

命名依据：

- 一律按 `service/{terminal}/{module}` 拆分。
- 名字看接口模块，不看底层数据库模块。

### 3.4 domain

职责：

- 承载跨多个 service 可复用的内部业务能力。
- 承载不适合直接放在某个 terminal service 里的核心领域能力。
- 组织多个 repository 完成一个内部能力闭环。

应该放什么：

- `hmd`：房源主数据能力
- `listingprojection`：HMD 变更到 HPD read model 的投影
- `publishaccess`：publish 数据作用域和归属关系

不应该放什么：

- 只服务单一 terminal 单一模块的薄封装
- 纯 DTO 映射
- 通用 CRUD

边界说明：

- domain 不是“所有业务都更高级地放一层”。
- 只有当逻辑已经脱离某个前端接口模块，变成内部可复用能力时，才应该进入 domain。
- 如果逻辑只属于 `service/miniapp/auth`，那它就留在 `service/miniapp/auth`，不必硬拆 `domain/*`。

命名依据：

- 按内部能力命名，不按 terminal 命名。
- 名字应表达能力本身，例如 `hmd`、`listingprojection`、`publishaccess`。

### 3.5 repository

职责：

- 承接数据库读写。
- 封装 collection 查询条件、字段白名单、软删除过滤、索引初始化。
- 在写入前调用 model validation。

应该放什么：

- `FindByID`、`FindOne`、`FindMany`
- `UpdateFieldsByID`
- `UpsertByUserID`
- Mongo filter / sort / index 定义

不应该放什么：

- 业务权限判断
- 终端级状态流转
- “当前登录用户是否允许操作该数据”这类规则

边界说明：

- repository 关心“怎么查、怎么写、哪些字段允许写”。
- repository 不关心“这次为什么要写、这个写入是否符合业务流程”。
- repository 可以返回 model，但不负责组装 API response。

命名依据：

- repository 按底层数据子域分包，不按 handler/service 的接口模块分包。
- 包名优先表达“操作的是哪类 collection / 实体”，例如 `landlord`、`hmd`、`hpd`。
- 一个 repository 包内可以包含多个 collection repository 文件，但这些 collection 必须共同属于一个稳定数据子域。
- 禁止用 `publish_auth`、`miniapp_auth` 这类上层业务入口名给 repository 命名。

为什么这样命名：

- repository 关心的是“底层数据怎么查、怎么写”，不是“哪个前端接口在调用它”。
- `service/publish/auth`、`service/miniapp/auth` 可以都是 `auth`，因为那是业务入口。
- 但 repository 应该落回实体/collection 语义，例如 `landlord`、`user`、`staff`，这样 service 才能明确看出自己依赖了哪些底层数据对象。

### 3.6 model

职责：

- 表达 Mongo collection 对应的数据结构。
- 定义枚举和值域方法。
- 定义单个 model 的落库结构不变量。

应该放什么：

- collection 常量
- struct 字段定义
- enum 定义
- `ValidateForCreate`
- `ValidateHmdUpdateFields`

不应该放什么：

- HTTP DTO
- 终端级业务规则
- 权限逻辑
- 跨 collection 一致性判断

边界说明：

- model 是数据库视角，不是接口视角。
- model 可以覆盖多个 terminal 共用的数据库模块，例如 `auth` 模块里同时存在用户和员工相关 collection。
- model validation 只校验“这个文档本身能不能安全落库”，不校验“这次业务操作是否合理”。

命名依据：

- model 按数据库模块分包。
- 每个数据库模块目录固定为：

```text
internal/model/{db-module}/
  model.go
  enum.go
  validation.go
```

- `model.go`：集合常量、struct
- `enum.go`：枚举、值域 helper
- `validation.go`：单个 Mongo model 的落库不变量

### 3.7 pkg

职责：

- 放基础设施和通用能力。
- 为 `repository`、`service`、`middleware` 等提供可复用工具。

应该放什么：

- `pkg/database/mongo`
- `pkg/database/redis`
- `pkg/session`
- `pkg/logger`
- `pkg/response`
- `pkg/errcode`

不应该放什么：

- 面向某个具体业务模块的逻辑
- 依赖 `internal/config` 的业务实现

边界说明：

- `pkg` 是横向基础设施层，不表达某个业务域。

## 4. 命名坐标系

项目里至少有两套命名坐标系，必须分开看：

### 4.1 接口坐标系

用于 `handler`、`service`、`repository`。

看的是：

- 哪个 terminal
- 哪个接口模块

例子：

- `handler/v1/miniapp/auth`
- `service/miniapp/auth`
- `repository/miniapp_auth`

### 4.2 数据库坐标系

用于 `model`。

看的是：

- 哪个数据库模块
- 哪些 collection 在同一个数据库模块下

例子：

- `model/auth`
- `model/hmd`
- `model/hpd`

结论：

- `model/auth` 存在，不代表 `repository` 也应该存在一个混合 `auth` 包。
- `repository` 和 `model` 可以使用不同命名依据，这是设计要求，不是例外。

## 5. 当前目录口径

### 5.1 已确认的 terminal 模块

- `miniapp/auth`
- `miniapp/house`
- `miniapp/favorite`
- `miniapp/history`
- `miniapp/user`
- `publish/auth`
- `publish` 下第一阶段 HMD 相关模块

### 5.2 已确认的 repository 分包

- `repository/miniapp_auth`
  - `user`
  - `user_auth`
  - `user_profile_ext`
- `repository/publish_auth`
  - `user`
- `repository/favorite`
- `repository/history`
- `repository/hmd`
- `repository/hpd`

### 5.3 已确认的 domain

- `domain/hmd`
- `domain/listingprojection`
- `domain/publishaccess`

## 6. 判定规则

遇到“这段代码应该放哪层”的问题时，按下面顺序判断：

1. 它是不是 HTTP 请求解析、DTO 校验、响应拼装？
   那就放 `handler`。

2. 它是不是某个 terminal 某个接口模块的业务编排、权限、作用域、状态流转？
   那就放对应 `service/{terminal}/{module}`。

3. 它是不是脱离单一 terminal、可被多个 service 复用的内部业务能力？
   那就放 `domain/{capability}`。

4. 它是不是 Mongo 读写、字段白名单、索引、软删除过滤？
   那就放 `repository/{module-or-terminal_module}`。

5. 它是不是 collection struct、枚举、落库不变量？
   那就放 `model/{db-module}`。

## 7. 禁止事项

- 不做旧系统兼容。
- 不把 handler 做成业务层。
- 不让 repository 承载跨业务流程。
- 不把多个业务域塞进一个大文件。
- 不把三端应用服务强行复用；miniapp、publish、admin 应按端侧拆 service。
- 不把不同 terminal 的同名接口模块混进同一个 repository 包。
- 不因为底层都属于同一个数据库模块，就强行把不同上层模块塞进一个 repository 包。
- 不在 model 里放 HTTP DTO、权限逻辑、状态流转。
