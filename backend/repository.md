# Backend Repository

## 约定

- 每个 collection 对应独立 repository 文件。
- 集合较多时按模块建目录，例如 `internal/repository/hmd`。
- 通用 CRUD 放在 `internal/repository/common`。
- `FindByID` 默认不返回软删除数据。
- `UpdateBaseInfo` 必须使用字段白名单。
- `UpdateStatus` 只更新状态字段。
- `UpsertFields` 必须防御空 filter。

## 实现补充

- [../schema/backend-crud/index.md](../schema/backend-crud/index.md)
