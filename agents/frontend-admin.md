# Frontend Admin Agent

## 必读

1. [global.md](./global.md)
2. [../product/admin.md](../product/admin.md)
3. [../api/index.md](../api/index.md)
4. [../schema/index.md](../schema/index.md)

## 工作规则

- 后台管理系统和发房系统是不同前端，不共享 handler 语义。
- 后台管理可以复用底层后端能力，但 API 需要按后台业务动作单独定义。
- 不直接假设发房系统 API 就等于后台管理 API。
- 审核、运营、数据维护类动作必须先在 product / api 中明确口径。
