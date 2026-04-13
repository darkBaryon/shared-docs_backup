# House Schema

## House（租客可见核心字段）

- `house_id`
  - 类型：`string`
  - 含义：房源 `_id` 的字符串形式。
- `area`
  - 类型：`string`
  - 含义：行政区。
- `location`
  - 类型：`string`
  - 含义：小区或街道。
- `type`
  - 类型：`string`
  - 含义：兼容展示字段，由 `room_count + hall_count` 派生，例如“两室一厅”。
- `price`
  - 类型：`integer`
  - 含义：月租金。
- `rent_mode`
  - 类型：`string`
  - 含义：出租方式，如 `whole` / `shared`。
- `room_count`
  - 类型：`integer`
  - 含义：卧室数量。
- `hall_count`
  - 类型：`integer`
  - 含义：客厅数量。
- `bathroom_count`
  - 类型：`integer`
  - 含义：卫生间数量。
- `payment_cycle`
  - 类型：`string`
  - 含义：付款方式，如 `monthly` / `quarterly`。
- `near_subway`
  - 类型：`boolean`
  - 含义：是否近地铁。
- `walk_to_subway_min`
  - 类型：`integer`
  - 含义：步行到地铁时间，分钟。
- `pet_friendly`
  - 类型：`boolean`
  - 含义：是否可养宠。
- `has_elevator`
  - 类型：`boolean`
  - 含义：是否有电梯。
- `cooking_allowed`
  - 类型：`boolean`
  - 含义：是否可做饭。
- `furnished_level`
  - 类型：`string`
  - 含义：装修/配套等级。
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
- `type`: `string`，当前仍为兼容查询字段，空字符串表示不限。
- `tags`: `string[]`，空数组表示不按标签过滤。
- `keyword`: `string`，空字符串表示不按关键词过滤。
- `page`: `integer`，页码。
- `limit`: `integer`，每页条数。

## 当前状态说明

- 租客侧 `search/public_detail/favorite/history` 已按新版可见字段联调。
- 房东侧 `create/update` 仍沿用旧写入字段 `type`，后续会单独切到新版写模型。
