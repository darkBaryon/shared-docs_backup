# POST /api/v1/centralized_room/list_by_project

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/room.(*Handler).ListCentralizedByProject`
- service：`internal/service/publish.(*centralizedRoomService).ListCentralizedRoomsByProject`

## 2. 处理逻辑

1. handler 绑定 `project_id`。
2. service 构造 `PublishScope`。
3. 校验当前房东是否可访问 `project_id`。
4. 通过后按 `project_id` 查询 `hs_hmd_room_centralized`。
5. 返回该项目下全部房间。
6. 当前实现里，无权访问父级时返回空数组 `[]`。

## 3. 数据读写

读取：

- `hs_hpd_root_scope_relation`
- `hs_hmd_room_centralized`

写入：

- 无

## 4. 失败返回

- `project_id` 非法：`InvalidParam`
- scope 查询失败：`DatabaseError`
- 房间查询失败：`DatabaseError`

## 5. 关键点

- 单父级查询，先 scope 再查 HMD
- 当前不走 `hs_hpd_publisher_listing` 作为主读取源
