# 当前进度

本文档只记录已经确认完成的事项，以及仍然应视为“未完成”的部分。

## 1. 当前阶段判断

当前已经完成：

- 配置和基础设施接入
- Auth 主链路跑通
- HMD repository 层落地
- 测试数据和索引脚本
- 发房系统接口命名和文档草案

当前尚未完成：

- publish 域整体架构定稿
- publish 上层 service
- publish handler / 路由
- HPD 实现
- 后台管理系统实现

所以当前阶段应定义为：

> 底层基础已经稳定，但发房系统业务实现还没有开始进入正式落地阶段。

## 2. 已确认完成事项

### 2.1 配置与基础设施

已完成：

- `internal/config` 已打通 YAML / `.env` / 环境变量读取
- `pkg/database/mongo` 已具备独立配置、连接和事务能力
- `pkg/database/redis` 已可用于 session 和缓存

当前判断：

- 这一层已经足够支撑后续模块继续接入

### 2.2 Auth 模块

已完成：

- 登录链路已端到端验证
- Redis session 写入和读取验证通过
- Bearer token 鉴权链路验证通过
- `auth/session` 受保护接口验证通过
- 注册链路已补 Mongo 事务

当前判断：

- Auth 主链路已可继续复用
- Auth 唯一索引目前仍然主要依赖脚本初始化，不是运行时自动保证

### 2.3 测试数据与初始化脚本

已完成：

- Auth 三张表 seed 脚本
- HMD 六张表 seed 脚本
- HMD 索引初始化脚本

当前判断：

- 服务器 Mongo 测试数据已经可以准备
- 现有数据结构已经足够支撑下一阶段开发

### 2.4 HMD 模型与 Repository

已完成集合：

- `hs_hmd_centralized`
- `hs_hmd_building`
- `hs_hmd_decentralized`
- `hs_hmd_room_type_centralized`
- `hs_hmd_room_centralized`
- `hs_hmd_room_decentralized`

已完成能力：

- 每个 collection 独立 repository 文件
- `FindByID`
- 业务唯一键查询
- 列表查询
- `UpdateBaseInfo`
- `UpdateStatus`

已完成的加固：

- `FindByID` 默认不返回软删除数据
- `UpdateBaseInfo` 有字段白名单
- `room_status` 独立更新并带合法性校验
- `common` 层不再污染调用方传入的 `bson.M`
- `UpsertFields` 已补空 filter 防御

当前判断：

- HMD repository 已足够给上层 service 直接复用

### 2.5 文档和命名约定

已完成：

- 发房系统 API 模块名确定为 `publish`
- 路径命名确定为 `POST /api/v1/publish/{action}`
- 发房系统第一期接口草案已写入共享文档

当前判断：

- 文档层已经明确 `publish` 是业务域，不是纯 HMD CRUD 模块

## 3. 仍然应视为未完成的部分

### 3.1 Publish 代码还不能算完成

当前仓库里已经有：

- `internal/service/publish/dto.go`
- `internal/service/publish/hmd_service.go`

但这两部分现在只能算：

- 草稿实现
- 未经过完整架构 review
- 不能当成 publish 域已经完成

原因：

- `publish` 整体设计还没最终定稿
- HPD 还没有进入 publish 架构
- handler、路由、请求结构、响应结构都还没形成完整链路
- 当前 `hmd_service.go` 已经承载了过多对象和校验逻辑，文件结构本身不可继续扩张
- 当前 `dto.go` 也已经把多类输入结构堆在一个文件里，边界不清

所以这部分代码要归类为：

> 已有草稿，但仍属于接下来要做的事情。

更准确地说：

> 当前 `internal/service/publish` 的文件结构本身就是阻塞项，在继续实现 publish 上层 service、handler、HPD 之前，必须先整理。

### 3.2 Publish 上层 service 未完成

尚未完成：

- 真正作为 publish 域入口的上层编排 service

当前判断：

- 现在只有 HMD 子能力草稿
- 真正的 publish use case 层还没开始正式落地

### 3.3 Publish 子 service 文件结构未完成

尚未完成：

- `internal/service/publish` 的文件级职责拆分

当前问题：

- `hmd_service.go` 已经同时承载：
  - 集中式项目
  - 楼栋
  - 房型
  - 集中式房间
  - 分散式小区
  - 分散式房间
  - 校验 helper
  - DTO 转换 helper
- `dto.go` 已经同时承载：
  - centralized
  - decentralized
  - room type
  - room
  - status
  - geo / image 辅助结构

当前判断：

- 这个结构不能继续往上叠 publish 上层 service、handler、HPD
- 否则 publish 域会在 very early stage 变成不可维护的大文件集合

### 3.4 Publish handler 与路由未完成

尚未完成：

- `internal/handler/v1/publish`
- publish 路由注册
- 应用装配链中的 publish 接入

当前判断：

- 发房系统还不能视为一个可调用的 API 模块

### 3.5 HPD 未完成

尚未完成：

- `hs_hpd_listing`
- `hs_hpd_listing_index`
- 录房动作到展示层数据的同步逻辑

当前判断：

- 这是 publish 域后续必须接入的部分，不是可无限后置的事情

## 4. 当前可以直接复用的基础

以下部分可以作为下一阶段稳定基础：

- `internal/repository/common`
- `internal/repository/auth`
- `internal/repository/hmd`
- `internal/service/auth`
- `api/发房系统接口文档.md`
- `schema/backend-crud.md`
- `schema/db-design/v4/*`

## 5. 当前主要风险

### 5.1 过早把 publish 草稿代码当成定稿

现在最大的风险，是直接在当前 `publish` 草稿上继续堆 handler 和 route，而没有先把整体架构评审清楚。

这样后面 HPD 接入时很容易推倒重来。

### 5.2 过早接受当前 publish 文件结构

现在不只是“publish 业务边界还没定稿”，还包括：

- 当前 `hmd_service.go` 已经太大
- 当前 `dto.go` 已经没有清晰边界

如果不先做文件级重组，后面的所有 publish 功能都会继续堆到这些文件上。

### 5.3 把 publish 误做成 HMD 直通层

如果下一步直接让 handler 调 HMD repository 或者只做 HMD CRUD 接口，`publish` 会被错误收窄成数据录入层。

这不是目标形态。

### 5.4 HPD 被拖得太后

发房系统最终不是只写 HMD。

它后面还要产出：

- 房东端展示数据
- 小程序展示数据
- 后台展示数据

所以 HPD 虽然可以晚一步实现，但不能在架构上缺席。
