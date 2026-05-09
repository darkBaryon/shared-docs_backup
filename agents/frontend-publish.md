# Frontend Publish Agent

## 必读

1. [global.md](./global.md)
2. [../product/publish.md](../product/publish.md)
3. [../api/publish.md](../api/publish.md)
4. [../schema/db-design/v4/modules/house-master-data.md](../schema/db-design/v4/modules/house-master-data.md)
5. [../flows/publish-listing.md](../flows/publish-listing.md)

## 工作规则

- 发房系统 API 固定使用 `POST /api/v1/publish/{action}`。
- 发房系统不是 HMD 表单直通层；录入动作后续会同时影响 HMD 和 HPD。
- 当前分散式房间没有房型模型，不传 `room_type_id`。
- 表单枚举值以 schema 和 API 文档为准，不在前端自造值。
- 新增发房接口前，先更新发房系统对应 API 文档。
