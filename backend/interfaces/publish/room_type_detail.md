# POST /api/v1/room_type/detail

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/roomtype.(*Handler).Detail`
- service：`internal/service/publish.(*roomTypeService).GetRoomType`

## 2. 处理逻辑

1. handler 绑定 `id`。
2. service 构造 `PublishScope`。
3. 读取 `hs_hmd_room_type_centralized`。
4. 如果 room type 带 `project_id`，按 project scope 校验。
5. 否则先读 `building`，再按其 `project_id` 校验。
6. 通过后返回；无权访问按 not found 处理。

## 3. 数据读写

读取：

- `hs_hmd_room_type_centralized`
- `hs_hmd_building`（必要时）
- `hs_hpd_root_scope_relation`

写入：

- 无

## 4. 失败返回

- `id` 非法：`InvalidParam`
- room type 不存在：`NotFound`
- 当前房东无权访问：`NotFound`
- 查询失败：`DatabaseError`
