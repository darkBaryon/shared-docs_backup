# Backend Validation

## 文件结构

model 按数据库模块分包组织。一个已落地数据库模块使用一个目录：`model.go` 放集合常量和结构，`enum.go` 放枚举和值域方法，`validation.go` 放落库结构不变量。

```text
internal/model/
  common/
    model.go
  auth/
    model.go
    enum.go
    validation.go
  hmd/
    model.go
    enum.go
    validation.go
  hpd/
    model.go
    enum.go
    validation.go
  useractivity/
    model.go
    enum.go
    validation.go
```

`internal/model/auth/model.go` 对齐账号与身份数据库模块，包含不同端的账号、认证、角色、权限等 collections，例如 `hs_usr_*` 和 `hs_adm_*`。

后续新增 HAC、Lead、Ops 等数据库模块时，新增对应模块文件。

## 规则

- 字段类型、集合常量放在对应数据库模块的 `model.go` 中。
- 枚举定义和值域方法放在对应模块的 `enum.go`。
- 单个 Mongo model 的落库结构不变量放在对应模块的 `validation.go`，例如枚举取值、必填字段、非负数。
- repository 写入前调用 model validation，但不承载业务规则。
- handler 只处理 HTTP 请求绑定、基础类型转换和传输层 DTO 校验。
- service/domain 负责业务规则、权限、数据作用域、状态流转和跨集合一致性。
- 不把多个数据库模块的 validation 写进一个大文件。
