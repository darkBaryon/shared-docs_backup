# 管理后台后端详细开发计划

本文档只解决两件事：

1. 每个 admin 模块要实现哪些接口
2. 每个接口具体落到哪些 Go 文件

前置说明：

- 当前 schema 未完成，因此本文档现在是“可执行骨架”，不是“已可直接编码”状态。
- `auth` 是当前唯一保留 terminal 前缀的模块，其它 admin 模块按 `<module>/<action>` 命名，不加 terminal 前缀。

## 1. 后端目标目录

第一阶段后端目标目录固定为：

```text
internal/handler/v1/admin/
  auth/
    handler.go
    dto.go
  staff/
    handler.go
    dto.go
  role/
    handler.go
    dto.go
  provider/
    handler.go
    dto.go
  house/
    handler.go
    dto.go
  launchaudit/
    handler.go
    dto.go
  handler.go
  routes.go

internal/service/admin/
  auth/
    service.go
    types.go
  staff/
    service.go
    types.go
  role/
    service.go
    types.go
  provider/
    service.go
    types.go
  house/
    service.go
    types.go
  launchaudit/
    service.go
    types.go
```

说明：

- `handler.go`：聚合依赖与对外方法
- `dto.go`：HTTP request / response DTO
- `service.go`：业务动作实现
- `types.go`：service input / output 类型

不新增 `service/admin/service.go` 大总管，避免过早聚合。

## 2. app 接线

需要新增：

```text
internal/handler/v1/admin/handler.go
internal/handler/v1/admin/routes.go
```

并在：

```text
internal/app/app.go
```

里注册 admin route group。

## 3. Phase 1：auth / staff / role

### 3.1 `admin_auth`

接口：

- `POST /api/v1/admin_auth/login`
- `POST /api/v1/admin_auth/logout`
- `POST /api/v1/admin_auth/session`
- `POST /api/v1/admin_auth/reset_password`

#### 文件

```text
internal/handler/v1/admin/auth/handler.go
internal/handler/v1/admin/auth/dto.go
internal/service/admin/auth/service.go
internal/service/admin/auth/types.go
```

#### `login`

请求：

- `phone`
- `password`

处理链：

```text
handler/admin/auth
  -> service/admin/auth.Login
    -> repository/staff.FindByPhone
    -> repository/role / permission
    -> session.Store.CreatePrincipal
```

实现要点：

- 主体是 `staff`
- `terminal=admin`
- 不复用 `publish_auth`
- 返回 token + principal 摘要 + 权限码列表

#### `session`

请求：

- 无 body 或空对象

处理链：

```text
middleware auth
  -> service/admin/auth.Session
    -> repository/staff.FindByID
    -> repository/role / permission
```

实现要点：

- 只接受 `terminal=admin`
- principal 必须是 `staff`

#### `logout`

处理链：

```text
handler -> service/admin/auth.Logout -> session.Store.Delete
```

#### `reset_password`

第一阶段后移，不进入首批实现。

### 3.2 `staff`

接口：

- `list`
- `detail`
- `create`
- `update`
- `disable`

#### 文件

```text
internal/handler/v1/admin/staff/handler.go
internal/handler/v1/admin/staff/dto.go
internal/service/admin/staff/service.go
internal/service/admin/staff/types.go
```

#### `list`

筛选项：

- 姓名关键词
- 手机号
- 角色
- 状态

处理链：

```text
handler -> service/admin/staff.List -> repository/staff.List
```

返回：

- 员工基础信息
- 角色名称列表
- 创建时间
- 登录时间

#### `detail`

返回：

- 员工基础信息
- 角色列表
- 创建人
- 登录信息

#### `create`

请求：

- `name`
- `phone`
- `password`
- `contact_qr`
- `role_ids`

处理链：

```text
handler
  -> service/admin/staff.Create
    -> repository/staff.Create
    -> repository/staff_role.ReplaceByStaffID
```

#### `update`

可修改：

- `name`
- `contact_qr`
- `role_ids`
- `status`

不可修改：

- `phone`

#### `disable`

作用：

- 员工停用
- 不物理删除

### 3.3 `role`

接口：

- `list`
- `detail`
- `create`
- `update`

#### 文件

```text
internal/handler/v1/admin/role/handler.go
internal/handler/v1/admin/role/dto.go
internal/service/admin/role/service.go
internal/service/admin/role/types.go
```

#### `list`

返回：

- 角色名称
- 权限摘要
- 创建时间

#### `detail`

返回：

- 角色基础信息
- 权限码完整列表

#### `create/update`

请求：

- `role_id`（仅 `update` 必填）
- `name`
- `permission_codes`

规则：

- 至少 1 个权限
- 权限码白名单由后端维护

## 4. Phase 2：provider

接口：

- `list`
- `detail`
- `create`
- `update`
- `disable`

#### 文件

```text
internal/handler/v1/admin/provider/handler.go
internal/handler/v1/admin/provider/dto.go
internal/service/admin/provider/service.go
internal/service/admin/provider/types.go
```

#### `list`

筛选项：

- 城市
- 发房方名称
- 手机号
- 状态

#### `detail`

返回：

- 基础信息
- 资质图片
- 品牌信息
- 房源概况摘要

#### `create`

请求：

- 城市
- 发房方名称
- 手机号码
- 门牌照图片
- 外观照图片
- 凭证资料图片

处理链：

```text
service/admin/provider.Create
  -> repository/landlord.Create
  -> repository/landlord_auth.CreatePasswordAuth
```

说明：

- `provider_id = hs_lld_landlord._id`
- 当前阶段只创建 `hs_lld_landlord + hs_lld_auth`
- `hs_lld_profile` 已有 schema 位置，但暂不进入实现
- 不把发房方资料硬塞进 `hs_usr_user` 或 `hs_lld_landlord`

#### `update`

允许修改：

- 名称
- 城市
- 手机号
- 资料图片

#### `disable`

只禁用账号，不删除。

## 5. Phase 3：house

接口：

- `list`
- `detail`
- `export`

#### 文件

```text
internal/handler/v1/admin/house/handler.go
internal/handler/v1/admin/house/dto.go
internal/service/admin/house/service.go
internal/service/admin/house/types.go
```

#### `list`

筛选项：

- 系统来源
- 发房方
- 项目
- 房源类型
- 城市 / 区域 / 商圈 / 地铁
- 审核状态
- 上下架状态

数据来源：

- 以 HMD + HPD 为主
- admin 是全量视角，不走 publish owner scope

#### `detail`

返回：

- 发房方信息
- 项目 / 楼栋 / 小区
- 房型 / 房间
- 当前状态
- 审核状态

#### `export`

第一阶段建议保留接口名，但实现优先级低于 `list/detail/update_*`。

说明：

- 第一阶段 `house` 只做运营视角查看，不做后台直接改服务信息或联系人。

## 6. Phase 4：launch_audit

接口：

- `list`
- `detail`
- `approve`
- `reject`

#### 文件

```text
internal/handler/v1/admin/launchaudit/handler.go
internal/handler/v1/admin/launchaudit/dto.go
internal/service/admin/launchaudit/service.go
internal/service/admin/launchaudit/types.go
```

#### `list`

查看：

- 待上架审核对象
- 审核状态

#### `detail`

返回：

- 审核单基础信息
- 当前生效内容
- 待审核内容
- 差异摘要

#### `approve/reject`

作用：

- 审核结果写回审核单
- 审核通过时更新目标对象的生效状态

说明：

- 必须以审核单为中心
- 不允许直接对 HMD 主对象做“裸审核”
- 当前第一阶段不做变更审核

## 7. repository / model 缺口

第一阶段预计新增：

- `internal/repository/admin_auth` 或继续复用现有 staff/role/permission repository
- 发房方业务资料 repository
- 上架审核单 repository

第一阶段预计新增 model：

- 发房方业务资料模型
- 房源上架审核单模型

## 8. 验收顺序

严格按 phase 推进：

1. `admin_auth`
2. `staff`
3. `role`
4. `provider`
5. `house`
6. `launch_audit`

每个模块完成前，必须同步更新：

- `api/admin.md`
- `backend/admin.md`
- `workstreams/admin-web/status.md`
