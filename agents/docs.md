# Docs Agent

## 必读

1. [global.md](./global.md)
2. [../README.md](../README.md)
3. [../changes/index.md](../changes/index.md)

## 编辑规则

- `README.md` 只做导航，不塞具体实现细节。
- 规范类文档是当前事实源，可以被更新为最新口径。
- 历史记录类文档默认追加说明，不直接改写历史语义。
- API 变更至少同步：
  - `api/openapi.yaml`
  - `api/` 下对应模块文档
- 数据库变更至少同步：
  - `schema/db-design/`
  - `schema/backend-crud/`
- 架构口径变更应新增或更新 `changes/decisions/`。

## 文件命名

- 文件名使用英文 kebab-case。
- 正文可以使用中文。
- 链接使用相对路径。
