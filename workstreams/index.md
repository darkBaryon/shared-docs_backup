# Workstreams

本目录是各端的当前推进区，只记录正在交付中的状态、任务计划和 review report。

长期有效的产品、接口、数据库和工程规范不要写在这里，应直接更新对应规范文档：

- `overview/`：项目背景、系统组成、术语。
- `product/`：产品范围和业务边界。
- `api/`：接口契约。
- `schema/`：数据库字段、枚举、索引、后端 CRUD。
- `backend/`：Go 后端工程约定。
- `frontend/`：前端工程约定。
- `flows/`：跨端业务流程。

## 目录规则

每个端使用相同结构：

```text
workstreams/{app}/
  index.md
  status.md
  active/
  reviews/
```

- `index.md`：该端工作流入口。
- `status.md`：当前状态总览，保持短文档，只放事实、入口和下一步。
- `active/`：正在推进的任务计划，一个任务一个文件。
- `reviews/`：review report，一个 report 一个文件。

不设置 `decisions/`。已经确认的技术、产品、接口或数据口径，应直接沉淀到对应规范文档；只影响当前任务的短期结论写入相关 `active/` 计划。

## 当前工作流

- [backend-go](./backend-go/index.md)：Go 后端迁移和后端能力建设。
- [publish-web](./publish-web/index.md)：出房 Web 前端。
- [miniapp](./miniapp/index.md)：小程序前端。
