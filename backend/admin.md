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

## 测试约定

admin 后端测试按分层落，不要求一开始把所有测试文件拆碎，但要求按风险覆盖关键行为。

### 基本原则

- 先保证每个对外接口有最小测试闭环，再扩展测试深度。
- 单测优先覆盖对外契约、关键业务分支、鉴权/权限边界、依赖失败分支。
- 纯薄 DTO 或无分支的转发层，不单独堆大量测试。
- 集成测试只验证主链路，不把所有分支都搬到集成测试里重测一遍。

### 文件组织

- 小模块起步可以只保留：
  - `handler_test.go`
  - `service_test.go`
- 当单个测试文件明显过长，或已经同时覆盖多个动作，再按动作拆成：
  - `login_test.go`
  - `session_test.go`
  - `logout_test.go`
- 只有公共样板重复明显时，才新增：
  - `helpers_test.go`
  - `fakes_test.go`
- 真实 Mongo / Redis 的集成测试统一放在仓库根目录：
  - `tests/integration/...`

### 覆盖标准

每个 admin 接口至少覆盖：

1. 成功路径
2. 参数错误
3. 未登录 / 登录态错误
4. 业务拒绝
5. 依赖失败

优先级最高的测试对象：

- 鉴权
- 权限
- 数据作用域
- 状态流转
- 缓存 / 数据库一致性
- 历史 bug 回归

### `admin_auth` 当前最低测试要求

- `handler`：
  - JSON 绑定成功
  - JSON 格式错误
  - service 错误透传
  - 响应结构正确
- `service`：
  - 登录成功
  - 手机号不存在 / 密码错误
  - session 恢复成功
  - principal 非 admin staff
  - logout 删除 token
  - Mongo / Redis 异常
- `middleware`：
  - 只接受 `terminal=admin`
  - token 缺失 / 失效
  - principal 注入 context
- `integration`：
  - `login -> session -> logout -> session fail`

当前 `admin_auth` 集成测试默认跳过，显式开启：

```bash
ADMIN_AUTH_INTEGRATION=1 \
go test ./tests/integration/admin_auth -run Integration -count=1 -v
```

可选覆盖测试环境：

```text
ADMIN_AUTH_TEST_DB
ADMIN_AUTH_TEST_ADDRS
ADMIN_AUTH_TEST_AUTH_SOURCE
ADMIN_AUTH_TEST_USERNAME
ADMIN_AUTH_TEST_PASSWORD
ADMIN_AUTH_TEST_REDIS_ADDRS
ADMIN_AUTH_TEST_REDIS_PASSWORD
```

约束：

- 默认应指向测试库。
- 如果目标库名不包含 `test`，必须显式设置 `ADMIN_AUTH_TEST_ALLOW_NON_TEST_DB=1` 才允许执行。
