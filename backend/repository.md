# Backend Repository

## 约定

- `repository` 目录按上层接口模块分包，不按纯数据库模块分包。
- 模块名在全局唯一时，目录直接使用模块名，例如 `favorite`、`history`、`hmd`、`hpd`。
- 模块名在不同 terminal 冲突时，目录必须使用 `{terminal}_{module}`，例如 `miniapp_auth`、`publish_auth`。
- 禁止建立混合型 repository 包去同时承载多个 terminal 的同名模块。
- 一个 repository 包里可以放多个 collection repository，但这些 collection 必须共同服务同一个上层模块边界。
- 每个 collection 对应独立 repository 文件。
- 集合较多时按模块建目录，例如 `internal/repository/hmd`、`internal/repository/miniapp_auth`。
- 通用 CRUD 放在 `internal/repository/common`。
- `FindByID` 默认不返回软删除数据。
- `UpdateBaseInfo` 必须使用字段白名单。
- `UpdateStatus` 只更新状态字段。
- `UpsertFields` 必须防御空 filter。

## 当前口径

- `internal/repository/miniapp_auth`：小程序认证相关 collection，包含 `user`、`user_auth`、`user_profile_ext`。
- `internal/repository/publish_auth`：publish 房东登录相关 collection，当前只承载 `hs_usr_user` 的房东手机号查询。
- `internal/repository/favorite`、`history`：直接对应小程序用户行为接口模块。
- `internal/repository/hmd`、`hpd`：当前上层模块名本身已唯一，因此不额外加 terminal 前缀。

## 判定规则

- 如果一个 collection 只被某个 terminal 的某个模块使用，就跟那个模块走。
- 如果多个 collection 总是被同一个上层模块一起编排，就允许放进同一个 repository 包。
- 如果只是因为底层都属于同一个数据库大模块就想合包，这个理由无效。

## 实现补充

- [../schema/backend-crud/index.md](../schema/backend-crud/index.md)
