# Schema

本目录维护数据库设计、字段约束、索引、枚举和后端 CRUD 实现约定。

## 数据库设计

- [db-design/v4/index.md](./db-design/v4/index.md)

## 后端 CRUD

- [backend-crud/index.md](./backend-crud/index.md)

## 规则

- 字段新增、删除、类型变更必须同步数据库设计文档。
- 枚举值变化必须同步 schema、model enum 和 API 文档。
- repository 新增或变更方法必须同步 backend CRUD 文档。
