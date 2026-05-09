# Backend Architecture

Go 后端采用分层结构：

```text
router
  -> handler
    -> service
      -> repository
        -> pkg/database
```

## 职责

- `router`：路径注册和 middleware 组合。
- `handler`：HTTP 参数绑定、调用 service、返回统一响应。
- `service`：业务动作、跨 repository 编排、事务边界。
- `repository`：数据库读写、字段白名单、软删除过滤。
- `model`：结构、枚举、字段约束。
- `pkg`：基础设施，不依赖 `internal/config`。

## 约束

- 不做旧系统兼容。
- 不把 handler 做成业务层。
- 不让 repository 承载跨业务流程。
- 不把多个业务域塞进一个大文件。
