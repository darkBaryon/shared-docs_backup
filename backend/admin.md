# Admin Backend

后台管理端后端边界说明。

## 目标

管理后台是独立 terminal，后端按独立端侧应用服务实现：

```text
handler/v1/admin/*
  -> service/admin/*
```

## 边界

- 可以复用 `domain/*` 和 `repository/*`
- 不复用 `service/miniapp/*`
- 不复用 `service/publish/*`
- 不把 admin 员工登录挂到 `publish_auth`
- `auth` 是 admin 唯一保留 terminal 前缀的例外模块，因为 miniapp 已占用 `/auth/session`

## 第一阶段模块

- `service/admin/auth`
- `service/admin/staff`
- `service/admin/role`
- `service/admin/provider`
- `service/admin/house`
- `service/admin/launchaudit`

## 目标文件结构

```text
internal/handler/v1/admin/
  auth/handler.go
  auth/dto.go
  staff/handler.go
  staff/dto.go
  role/handler.go
  role/dto.go
  provider/handler.go
  provider/dto.go
  house/handler.go
  house/dto.go
  launchaudit/handler.go
  launchaudit/dto.go
  handler.go
  routes.go

internal/service/admin/
  auth/service.go
  auth/types.go
  staff/service.go
  staff/types.go
  role/service.go
  role/types.go
  provider/service.go
  provider/types.go
  house/service.go
  house/types.go
  launchaudit/service.go
  launchaudit/types.go
```

管理后台当前不引入新的大总管 `service/admin/service.go`，按模块直接组织。

## 职责

### `admin/auth`

- 员工登录
- session 恢复
- 密码重置

### `admin/staff`

- 员工增删改查
- 员工状态控制

### `admin/role`

- 角色增删改查
- 权限分配

### `admin/provider`

- 发房方资料管理
- 发房方账号启用 / 禁用

### `admin/house`

- 平台视角全量房源查询
- 标签、联系人、导出等运营动作

### `admin/launchaudit`

- 上架审核
- 审核结果落地

## 数据模型方向

第一阶段会补：

- 员工权限相关模型
- 发房方业务资料模型
- 上架审核单模型

## 实施顺序

1. `auth`
2. `staff`
3. `role`
4. `provider`
5. `house`
6. `launchaudit`

不直接把审核状态和审核备注散写在 HMD 主档上替代审核单。
