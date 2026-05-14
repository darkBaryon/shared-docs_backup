# Docs Agent

## 必读

1. [global.md](./global.md)
2. [../README.md](../README.md)
3. [../workstreams/index.md](../workstreams/index.md)

## 编辑规则

- `README.md` 只做导航，不塞具体实现细节。
- 规范类文档是当前事实源，可以被更新为最新口径。
- 历史记录类文档不长期保留；当前事实应直接更新到规范文档。
- API 变更至少同步 `api/` 下对应前端的端侧 API 文档。
- 数据库变更至少同步：
  - `schema/db-design/`
  - `schema/backend-crud/`
- 架构口径变更应更新对应规范文档和相关 `workstreams/{app}/status.md`。
- 具体任务计划写入 `workstreams/{app}/active/{topic}.md`，不要堆进 `status.md`。
- review report 写入 `workstreams/{app}/reviews/YYYY-MM-DD-topic.md`。
- 不新建 `decisions/`；已确认口径直接写入规范文档。

## 文件命名

- 文件名使用英文 kebab-case。
- 正文可以使用中文。
- 链接使用相对路径。
