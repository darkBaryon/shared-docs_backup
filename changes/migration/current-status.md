# 当前进度

本文档只记录已经确认完成的事项，以及仍然应视为“未完成”的部分。

## 1. 当前阶段判断

当前已经完成：

- 配置和基础设施接入
- Auth 主链路跑通
- HMD repository 层落地
- 测试数据和索引脚本
- 发房系统接口命名和文档草案
- publish service / handler / route 第一版接入
- shared-docs 文档模块化整理

当前尚未完成：

- publish 第一版接口端到端联调
- HPD 实现
- 后台管理系统实现

所以当前阶段应定义为：

> 底层基础已经稳定，publish 第一版已接入代码链路，下一步重点是联调、补测试和继续 HPD。

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

### 2.6 Publish 第一版代码链路

已完成：

- `internal/service/publish` 顶层已收敛为 facade
- HMD 子域已拆到 `internal/service/publish/hmd`
- HPD 子域已拆到 `internal/service/publish/hpd`
- `PublishService` 已作为 handler 入口
- `hmd.Service` 已作为 HMD 子 service
- `hpd.Service` 已作为 no-op 预留点
- HMD 写操作已返回 `Entity + Changes`
- PublishService 已统一调用 `hpd.Service.Apply(changes)`
- `internal/handler/v1/publish` 已接入第一期 publish API
- publish handler 已从 `internal/handler/v1` 根包移入独立子包
- publish handler request 已按对象拆分，不再把所有请求结构堆在单个大文件
- handler HTTP request 已和 service DTO 解耦，`geo/images` 等字段显式转换到 service input
- publish service 已返回明确 `errcode`，handler 不再用字符串归类 service 错误
- publish 路由已注册到受保护 `/api/v1` 路由组
- Wire provider 已接入 publish 依赖
- `internal/service/publish/hmd` 已补 Mongo 集成测试，默认跳过，显式开启后验证 HMD 主链路
- `internal/service/publish` 已补 facade 单元测试，验证 HMD mutation changes 会统一派发给 HPD Apply
- `internal/handler/v1/publish` 已补基础 HTTP binding 测试，覆盖 JSON 绑定、ObjectID 解析、参数错误和 service errcode 响应

当前判断：

- publish 已经不是早期大文件草稿
- HMD service、publish facade 和 handler 边界已开始分层验证
- 但还需要真实服务 curl 联调

## 3. 仍然应视为未完成的部分

### 3.1 Publish 还需要端到端验证

当前仓库里已经有：

- `internal/service/publish`
- `internal/handler/v1/publish`
- Wire publish provider

但这部分仍然不能直接视为完全完成：

- 还没有真实服务启动后的 curl 联调记录
- handler request binding 目前只有基础覆盖，还需要随接口扩展继续补充
- HPD projector / outbox 尚未实现
- HPD 当前 `Apply` 仍是 no-op

当前判断：

- publish 第一版链路已进入可联调状态
- 下一步应做真实服务 curl 联调和问题修复

### 3.2 HPD 未完成

尚未完成：

- `hs_hpd_listing`
- `hs_hpd_listing_index`
- 录房动作到展示层数据的同步逻辑

当前判断：

- 这是 publish 域后续必须接入的部分，不是可无限后置的事情

### 3.3 后台管理系统未完成

尚未完成：

- 后台管理产品范围
- 后台管理 API
- 后台管理 handler / service

当前判断：

- 后台管理和发房系统是两个前端，后续应独立设计 API 和 handler

## 4. 当前可以直接复用的基础

以下部分可以作为下一阶段稳定基础：

- `internal/repository/common`
- `internal/repository/auth`
- `internal/repository/hmd`
- `internal/service/auth`
- `internal/service/publish`
- `internal/handler/v1/publish`
- `api/publish.md`
- `schema/backend-crud/index.md`
- `schema/db-design/v4/*`

## 5. 当前主要风险

### 5.1 过早把 publish 第一版当成完全定稿

publish 第一版已经接入，但仍需要真实接口联调和测试补强。

如果不经过联调就继续堆 HPD 或后台管理，后续仍可能返工。

### 5.2 HPD 被拖得太后

发房系统最终不是只写 HMD。

它后面还要产出：

- 房东端展示数据
- 小程序展示数据
- 后台展示数据

所以 HPD 虽然可以晚一步实现，但不能在架构上缺席。

### 5.3 把后台管理误并入发房系统

后台管理和发房系统不是同一个前端，也不是同一组 handler。

后续后台管理应单独定义产品范围、API 和 service 入口。
