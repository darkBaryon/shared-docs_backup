# Backend Validation

## 文件结构

model 按业务域拆成三类文件：

```text
internal/model/
  hmd.go
  hmd_enum.go
  hmd_validation.go

  user.go
  user_enum.go
  user_validation.go

  auth.go
  auth_enum.go
  auth_validation.go
```

后续新增 HPD、HAC、Lead 时沿用同样结构。

## 规则

- 字段取值范围放到 enum 文件。
- create/update 约束放到 validation 文件。
- repository 写入前调用 model validation。
- handler 只处理 HTTP 请求绑定和基础类型转换。
- 不把所有业务域的 validation 写进一个大文件。
