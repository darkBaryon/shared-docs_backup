# 三端应用服务与 Handler 架构迁移计划

## 1. 背景

当前 Go 后端正在从 Python 后端迁移到 `house-manager`。

项目已经确认存在三个主要前端端侧：

- 用户小程序端
- 房东 / 发房 Web 端
- 管理后台端

三端虽然都会涉及用户、登录、房源、操作记录等概念，但入口、权限、展示语义、业务流程和接口契约差异很大。不能为了形式复用，把三端接口强行塞进同一套 service / handler 包中。

## 2. 核心结论

后续代码结构按“端侧应用服务 + 内部领域能力”拆分：

```text
internal/handler/v1/
  miniapp/
  publish/
  admin/

internal/service/
  miniapp/
  publish/
  admin/

internal/domain/
  hmd/
  hpd/
```

其中：

- `handler/v1/{terminal}` 是端侧 API 表达层。
- `service/{terminal}` 是端侧应用服务层。
- `domain/hmd`、`domain/hpd` 是内部领域能力层，不直接代表某个前端。
- `repository/*` 继续作为数据访问层。

## 3. 设计原则

### 3.1 Handler 按端侧拆

Handler 是 HTTP API 边界，应按前端端侧组织：

```text
internal/handler/v1/miniapp/
internal/handler/v1/publish/
internal/handler/v1/admin/
```

端侧下面再按 API 模块拆：

```text
internal/handler/v1/miniapp/
  auth/
  user/
  house/
  favorite/
  history/

internal/handler/v1/publish/
  auth/
  house/
  building/
  room_type/

internal/handler/v1/admin/
  auth/
  user/
  house/
  audit/
```

注意：代码目录里的 `miniapp` 不等于 URL 必须出现 `miniapp`。小程序 API 路由仍按接口契约保持：

```text
POST /api/v1/house/search
POST /api/v1/favorite/list
POST /api/v1/user/profile
```

### 3.2 Service 也按端侧拆

三端业务差异不是“少量 UI 差异”，而是应用流程、登录方式、权限、请求语义都不同。

因此 service 不再强行抽象成一套全局业务 service：

```text
internal/service/house
internal/service/user
internal/service/auth
```

推荐改为：

```text
internal/service/miniapp/
  auth/
  user/
  house/
  favorite/
  history/

internal/service/publish/
  auth/
  house/
  building/
  room_type/

internal/service/admin/
  auth/
  user/
  house/
  audit/
```

这样即使三端都有 `house` 或 `auth`，也不会互相污染：

```text
service/miniapp/house
service/publish/house
service/admin/house

service/miniapp/auth
service/publish/auth
service/admin/auth
```

### 3.3 HMD / HPD 迁到 domain

`hmd` 和 `hpd` 不是端侧应用服务。

- HMD 是房源主数据领域能力。
- HPD 是房源发布展示领域能力 / read model projection。

更合适的位置是：

```text
internal/domain/hmd
internal/domain/hpd
```

职责：

- `domain/hmd`：HMD 主数据校验、组合写入、领域变更输出。
- `domain/hpd`：HPD projection、lifecycle、`Apply(changes)`。

约束：

- `domain/hmd`、`domain/hpd` 不知道 `miniapp` / `publish` / `admin`。
- `handler` 不直接调用 `domain/hmd` / `domain/hpd`。
- 端侧 service 可以调用 domain。
- 小程序读接口仍只读 `repository/hpd.MiniappListingRepository`，不得调用 HPD projector。

## 4. 目标依赖链

### 4.1 小程序找房

```text
handler/v1/miniapp/house
  -> service/miniapp/house
    -> repository/hpd.MiniappListingRepository
      -> hs_hpd_miniapp_listing
```

### 4.2 小程序收藏

```text
handler/v1/miniapp/favorite
  -> service/miniapp/favorite
    -> repository/favorite
    -> repository/hpd.MiniappListingRepository
```

### 4.3 小程序足迹

```text
handler/v1/miniapp/history
  -> service/miniapp/history
    -> repository/history
    -> repository/hpd.MiniappListingRepository
```

### 4.4 发房端

```text
handler/v1/publish/{module}
  -> service/publish/{module}
    -> domain/hmd
    -> domain/hpd
    -> repository/hmd
    -> repository/hpd
```

### 4.5 管理端

```text
handler/v1/admin/{module}
  -> service/admin/{module}
    -> domain/hmd
    -> domain/hpd
    -> repository/admin / repository/hmd / repository/hpd
```

## 5. 当前代码迁移计划

### 5.1 第一阶段：小程序 house 结构归位

当前已有：

```text
internal/handler/v1/house
internal/service/house
```

需要迁到：

```text
internal/handler/v1/miniapp/house
internal/service/miniapp/house
```

路由保持不变：

```text
POST /api/v1/house/search
POST /api/v1/house/public_detail
```

Wire 注册同步调整为：

```text
MiniappHouseSet
```

### 5.2 第二阶段：HMD / HPD domain 化

当前已有：

```text
internal/service/hmd
internal/service/hpd
```

目标迁到：

```text
internal/domain/hmd
internal/domain/hpd
```

迁移后：

- publish 端 service 调用 `domain/hmd` 和 `domain/hpd`。
- miniapp house 读接口不调用 `domain/hpd`，继续只读 `repository/hpd`。
- 测试包路径同步迁移。

### 5.3 第三阶段：publish handler / service 按端侧整理

当前已有：

```text
internal/handler/v1/publish
internal/service/publish
```

短期可以保留，因为 `publish` 本身已经是端侧入口。

后续如果 publish 端继续膨胀，应从单包拆成：

```text
internal/handler/v1/publish/
  centralized_project/
  building/
  room_type/
  centralized_room/
  decentralized_community/
  decentralized_room/

internal/service/publish/
  centralized_project/
  building/
  room_type/
  centralized_room/
  decentralized_community/
  decentralized_room/
```

拆分时保持 URL 不变：

```text
POST /api/v1/publish/{action}
```

## 6. 禁止事项

- 禁止新增 `service/miniapp` 大包，把小程序所有接口塞进一个包。
- 禁止新增 `handler/v1/miniapp` 大包，把小程序所有 handler 塞进一个包。
- 禁止让 `handler` 直接调用 repository。
- 禁止让 `handler` 直接调用 `domain/hmd` 或 `domain/hpd`。
- 禁止让小程序 house 读接口调用 HPD projector。
- 禁止为了旧 Python、旧前端、旧数据库字段做兼容适配。

## 7. 文档维护规则

所有迁移计划统一放在：

```text
shared-docs/changes/migration/
```

计划完成后必须更新对应计划文档：

- 标记已完成事项。
- 写清实际落地路径。
- 如果实现与计划不同，补充原因和最终决策。
- 必要时同步更新 `current-status.md` 和 `next-steps.md`。

不要把迁移计划长期散落在聊天记录、临时文件、API 文档或 backend 规范文档里。
