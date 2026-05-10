# 当前迁移状态与下一步计划

本文是 Go 后端迁移的当前事实源。历史阶段性计划、过期调整记录和已废弃接口说明不再单独保留。

## 1. 当前项目怎么运作

后端分为三类代码：

```text
handler/v1/{terminal}/{module}
  -> service/{terminal}/{module}
    -> domain/{capability}
      -> repository/{data-module}
        -> MongoDB / Redis
```

含义：

- `handler/v1/{terminal}`：端侧 HTTP API 表达层，例如 `miniapp`、`publish`、`admin`。
- `service/{terminal}`：端侧应用服务。三端登录、权限、业务入口差异很大，不共用一个应用 service。
- `domain/*`：内部领域能力，不代表某个前端。
- `repository/*`：数据访问层。

当前已确认的内部领域：

```text
internal/domain/auth  // 微信身份、用户初始化、资料活跃时间维护
internal/domain/hmd   // 房源主数据领域能力
internal/domain/hpd   // 展示层 read model projection / lifecycle
```

当前已确认的端侧应用服务：

```text
internal/service/miniapp/auth
internal/service/miniapp/house
internal/service/publish
```

## 2. 当前已完成

### 2.1 基础设施

- 配置读取、Mongo、Redis 已接入。
- Redis session token 已用于 Bearer 鉴权。
- repository common 层、字段白名单、软删状态过滤已具备基础能力。

### 2.2 小程序 auth

已完成：

- `POST /api/v1/auth/wechat_login`
- `POST /api/v1/auth/wechat_register`
- `POST /api/v1/auth/session`

代码结构：

```text
handler/v1/miniapp/auth
  -> service/miniapp/auth
    -> domain/auth
      -> repository/auth
      -> Redis session
```

说明：

- token 是 opaque Redis session token，不是 JWT。
- 小程序 auth 不作为 publish/admin 的通用登录入口。
- 后续 publish/admin auth 需要分别建立自己的端侧应用服务。

### 2.3 Publish 第一阶段

已完成：

- publish handler / service / route 第一版接入。
- HMD 写操作返回 `HmdMutationResult{Entity, Changes}`。
- `service/publish` 统一把 changes 交给 `domain/hpd.Service.Apply(changes)`。
- publish 第一阶段 HMD 链路已通过真实服务联调。

代码结构：

```text
handler/v1/publish
  -> service/publish
    -> domain/hmd
    -> domain/hpd
    -> repository/hmd
    -> repository/hpd
```

### 2.4 HPD 小程序展示层

已完成：

- `hs_hpd_listing`
- `hs_hpd_miniapp_listing`
- HMD change -> HPD projector
- 小程序房源 read model mapper / fanout
- 小程序 house 读接口

小程序 house 链路：

```text
handler/v1/miniapp/house
  -> service/miniapp/house
    -> repository/hpd.MiniappListingRepository
      -> hs_hpd_miniapp_listing
```

已完成接口：

- `POST /api/v1/house/search`
- `POST /api/v1/house/public_detail`

约束：

- URL 保持 `/api/v1/house/*`，代码目录用 `miniapp` 表示端侧。
- house 读接口只读 `hs_hpd_miniapp_listing`，不直接读 HMD。
- handler 不直接调用 repository 或 domain projector。
- miniapp API 请求和响应字段统一使用 `snake_case`。

### 2.5 小程序用户资料、收藏、足迹

已完成：

```text
internal/model/user_activity.go

internal/repository/favorite
internal/repository/history

internal/service/miniapp/favorite
internal/service/miniapp/history
internal/service/miniapp/user

internal/handler/v1/miniapp/favorite
internal/handler/v1/miniapp/history
internal/handler/v1/miniapp/user
```

已完成接口：

- `POST /api/v1/user/profile`
- `POST /api/v1/user/update_profile`
- `POST /api/v1/user/dashboard`
- `POST /api/v1/favorite/add`
- `POST /api/v1/favorite/remove`
- `POST /api/v1/favorite/list`
- `POST /api/v1/history/add`
- `POST /api/v1/history/list`

落地口径：

- `hs_usr_favorite.listing_id` 关联 `hs_hpd_listing._id`。
- `hs_usr_history.listing_id` 关联 `hs_hpd_listing._id`。
- 小程序收藏列表和足迹列表只展示仍在线的房源，即只返回 `hs_hpd_miniapp_listing.is_online=1` 的展示快照。
- 收藏取消使用状态更新，不物理删除。
- 足迹重复浏览同一房源时更新 `viewed_at` 和 `source`。
- `source` 当前限制为 `discover` / `ai` / `detail`。
- `favorite/add` 幂等：重复收藏返回成功。
- `favorite/remove` 幂等：未收藏返回成功。
- `history/list` 按 `viewed_at` 倒序。
- `user/dashboard` 当前返回真实收藏数和足迹数；`plan_count`、`unread_notification_count` 为 `0`，等看房计划和通知模块实现。
- 小程序响应字段使用 `snake_case`。
- 分页响应为 `data.list/page/page_size/total`，不使用 `response.SuccessPage`。

索引规划：

```text
hs_usr_favorite
  user_id_1_listing_id_1 unique
  user_id_1_status_1_updated_at_-1

hs_usr_history
  user_id_1_listing_id_1 unique
  user_id_1_viewed_at_-1
```

`hs_usr_history.viewed_at` 的 90 天 TTL 暂不作为本阶段阻塞项；如果项目已有统一索引初始化入口，再一起补。

验证情况：

- `go test ./...` 通过。
- 真实服务联调已验证：`user/profile`、`user/update_profile`、`user/dashboard`、`favorite/add`、`favorite/list`、`favorite/remove`、`history/add`、`history/list`。

### 2.6 小程序 public_detail 可选登录

已完成：

- `POST /api/v1/house/public_detail` 仍是公开接口。
- 匿名访问返回 `is_favorited=false`。
- 带有效 token 时返回真实 `is_favorited`。
- 无效 token 按匿名处理，避免公开详情因过期 token 无法打开。
- 实现路径：
  - `internal/middleware/optional_auth.go`
  - `internal/service/miniapp/house` 通过窄接口查询 favorite 状态
  - `internal/handler/v1/miniapp/house` 只对 `public_detail` 挂 optional auth

验证情况：

- 真实服务联调已验证：收藏后详情 `is_favorited=true`，取消收藏后详情 `is_favorited=false`，匿名详情 `is_favorited=false`。

## 3. 当前未完成

### 3.1 管理端

后台管理端尚未开始实现。后续应独立设计：

后台管理端尚未开始实现。后续应独立设计：

```text
handler/v1/admin/*
service/admin/*
```

不能复用小程序或 publish 的端侧应用 service。

### 3.2 小程序后续非第一阶段模块

后续按业务优先级单独实现：

- 看房计划。
- 通知中心。

## 4. 当前重要文档

日常只需要先读这些：

1. [../../README.md](../../README.md)：项目文档入口和阅读路径。
2. [../../api/miniapp-api.md](../../api/miniapp-api.md)：小程序 API 契约。
3. [../../api/publish.md](../../api/publish.md)：发房端 API 契约。
4. [../../backend/architecture.md](../../backend/architecture.md)：Go 后端分层。
5. [../../backend/publish.md](../../backend/publish.md)：publish 后端链路。
6. [../../backend/miniapp-hpd.md](../../backend/miniapp-hpd.md)：小程序 HPD 展示层。
7. [../../schema/db-design/v4/index.md](../../schema/db-design/v4/index.md)：数据库设计入口。

如果这些文档和历史聊天记录冲突，以这些文档为准。

## 5. 文档维护规则

- 迁移状态只维护本文，不再新增一堆临时进度文件。
- 做完迁移任务后更新本文的“已完成 / 未完成 / 下一步”。
- 已废弃接口、旧路径、旧数据库字段说明应删除，不长期保留在 changes 里。
- 长期有效的架构口径直接写入对应规范文档，例如 `backend/architecture.md`、`backend/publish.md`、`backend/miniapp-hpd.md`。
