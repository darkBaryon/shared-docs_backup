# Backend Architecture

Go 后端采用分层结构：

```text
router
  -> handler/v1/{terminal}/{module}
    -> service/{terminal}/{module}
      -> domain/{capability}
      -> repository/{data-module}
        -> pkg/database
```

## 职责

- `router`：路径注册和 middleware 组合。
- `handler`：HTTP 参数绑定、鉴权上下文读取、调用端侧 service、返回统一响应。
- `service`：端侧应用服务，表达某个前端入口的业务动作和编排。
- `domain`：内部领域能力，例如 auth、HMD、HPD，不直接代表某个前端。
- `repository`：数据库读写、字段白名单、软删除过滤。
- `model`：结构、枚举、字段约束。
- `pkg`：基础设施，不依赖 `internal/config`。

## 约束

- 不做旧系统兼容。
- 不把 handler 做成业务层。
- 不让 repository 承载跨业务流程。
- 不把多个业务域塞进一个大文件。
- 不把三端应用服务强行复用；miniapp、publish、admin 应按端侧拆 service。
