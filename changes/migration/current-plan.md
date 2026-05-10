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

## 3. 当前未完成

### 3.1 小程序剩余接口

待实现：

```text
handler/v1/miniapp/user
service/miniapp/user

handler/v1/miniapp/favorite
service/miniapp/favorite

handler/v1/miniapp/history
service/miniapp/history
```

对应接口以 [../../api/miniapp-api.md](../../api/miniapp-api.md) 为准。

建议顺序：

1. `repository/favorite`
2. `repository/history`
3. `service/miniapp/favorite`
4. `service/miniapp/history`
5. `service/miniapp/user`
6. `handler/v1/miniapp/favorite`
7. `handler/v1/miniapp/history`
8. `handler/v1/miniapp/user`

原因：

- `user/dashboard` 需要收藏数和足迹数。
- `house/public_detail` 的真实 `is_favorited` 需要 favorite 能力。

### 3.2 小程序 public_detail 可选登录

当前 `house/public_detail` 是公开接口。

后续 favorite 完成后，需要支持：

```text
匿名访问：is_favorited=false
带有效 token：返回真实 is_favorited
```

实现时应新增 optional auth 逻辑，不能直接挂强制 auth middleware。

### 3.3 管理端

后台管理端尚未开始实现。后续应独立设计：

```text
handler/v1/admin/*
service/admin/*
```

不能复用小程序或 publish 的端侧应用 service。

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
