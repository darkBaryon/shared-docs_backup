# House Schema

## House（租客可见核心字段）

- `house_id`
  - 类型：`string`
  - 含义：房源唯一标识。
- `area`
  - 类型：`string`
  - 含义：行政区。
- `location`
  - 类型：`string`
  - 含义：小区或街道。
- `type`
  - 类型：`string`
  - 含义：户型（如两室一厅）。
- `price`
  - 类型：`integer`
  - 含义：月租金。
- `tags`
  - 类型：`string[]`
  - 含义：展示标签（近地铁、精装修等）。
- `images`
  - 类型：`string[]`
  - 含义：图片 URL 列表。
- `status`
  - 类型：`integer`
  - 枚举：`1`待租、`2`已租、`0`下架。
- `created_at`
  - 类型：`int64`
  - 含义：创建时间（秒级时间戳）。
- `updated_at`
  - 类型：`int64`
  - 含义：更新时间（秒级时间戳）。

## HouseSearchRequest

- `area`: `string`，空字符串表示不限。
- `min_price`: `integer`，`0` 视为未指定。
- `max_price`: `integer`，`0` 视为未指定。
- `type`: `string`，空字符串表示不限。
- `tags`: `string[]`，空数组表示不按标签过滤。
- `keyword`: `string`，空字符串表示不按关键词过滤。
- `page`: `integer`，页码。
- `limit`: `integer`，每页条数。
