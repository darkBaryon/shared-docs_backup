# POST /api/v1/decentralized_room/update_status

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/room.(*Handler).UpdateDecentralizedStatus`
- service：`internal/service/publish.(*decentralizedRoomService).UpdateDecentralizedRoomStatus`

## 2. 处理逻辑

1. handler 绑定 `id` 与 `room_status`。
2. service 构造 `PublishScope`。
3. 先读 room。
4. 用 `room.decentralized_id` 判断当前房东是否可操作。
5. 通过后更新 `hs_hmd_room_decentralized.room_status`。
6. HMD 返回 change。
7. service 调 `listingprojection.Apply(changes)` 刷新相关 HPD projection。
8. 返回更新后的 room。

## 3. 数据读写

读取：

- `hs_hmd_room_decentralized`
- `hs_hpd_root_scope_relation`

写入：

- `hs_hmd_room_decentralized.room_status`
- 相关 HPD projection

## 4. 失败返回

- `id` 或 `room_status` 非法：`InvalidParam`
- 房间不存在：`NotFound`
- 当前房东无权操作：`NotFound`
- HMD 更新失败：`DatabaseError`
- projection 刷新失败：内部错误 / projection error
