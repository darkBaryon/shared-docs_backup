# Backend Repository

## 约定

- `repository` 目录按底层数据子域分包，不按 handler/service 的接口模块分包。
- 包名优先表达“操作的是哪类 collection / 实体”，例如 `landlord`、`hmd`、`hpd`、`favorite`、`history`。
- 禁止建立混合型 repository 包去同时承载多个无关数据对象，也不要用 `publish_auth`、`miniapp_auth` 这种上层业务名给 repository 命名。
- 一个 repository 包里可以放多个 collection repository，但这些 collection 必须属于同一数据子域。
- 每个 collection 对应独立 repository 文件。
- 集合较多时按子域建目录，例如 `internal/repository/hmd`、`internal/repository/landlord`。
- 通用 CRUD 放在 `internal/repository/common`。
- `FindByID` 默认不返回软删除数据。
- `UpdateBaseInfo` 必须使用字段白名单。
- `UpdateStatus` 只更新状态字段。
- `UpsertFields` 必须防御空 filter。

## 当前口径

- `internal/repository/landlord`：房东身份与认证数据，承载 `hs_lld_landlord` / `hs_lld_auth` 的底层查询。
- `internal/repository/adm`：后台员工、角色、权限和登录审计数据，承载 `hs_adm_*` 集合的底层读写。
- `internal/repository/hmd`：房源主数据六个模块的底层读写。
- `internal/repository/hpd`：发布/展示域数据与 root owner scope relation。
- `internal/repository/favorite`、`history`：当前仍按独立 collection 能力维护，后续如果 user-behavior 子域扩大，再统一收目录。
- `internal/repository/miniapp_auth` 属于旧命名，后续应继续向实体/子域命名收敛，不再新增同类命名。

## 判定规则

- 如果多个 collection 共同表达一个稳定的数据子域，就进入同一个 repository 包。
- 如果只是某个 terminal 的某个接口暂时会同时调用几张表，不足以成为 repository 命名依据。
- 如果一个包名读起来像业务动作或前端入口，例如 `publish_auth`、`admin_auth`，通常说明它更适合放在 service，而不是 repository。

## 实现补充

- [../schema/backend-crud/index.md](../schema/backend-crud/index.md)
