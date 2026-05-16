# POST /api/v1/room_type/list_by_building

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/roomtype.(*Handler).ListByBuilding`
- service：`internal/service/publish.(*roomTypeService).ListRoomTypesByBuilding`

## 2. 处理逻辑

1. handler 绑定 `building_id`。
2. service 构造 `PublishScope`。
3. 先读 building，拿到 `project_id`。
4. 校验当前房东是否可访问该 project。
5. 通过后按 `building_id` 查 room type。
6. 当前实现里，无权访问父级时返回空数组 `[]`。

## 3. 数据读写

读取：

- `hs_hmd_building`
- `hs_hpd_root_scope_relation`
- `hs_hmd_room_type_centralized`

写入：

- 无

## 4. 失败返回

- `building_id` 非法：`InvalidParam`
- scope 查询失败：`DatabaseError`
- room type 查询失败：`DatabaseError`
