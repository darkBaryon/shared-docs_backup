# POST /api/v1/centralized_room/update

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/room.(*Handler).UpdateCentralized`
- service：`internal/service/publish.(*centralizedRoomService).UpdateCentralizedRoom`

## 2. 处理逻辑

1. handler 绑定房间更新字段。
2. service 构造 `PublishScope`。
3. 先读 room。
4. 用 `room.project_id` 判断当前房东是否可访问。
5. 通过后调用 `domain/hmd.UpdateCentralizedRoom` 更新 HMD。
6. HMD 返回 change。
7. service 调 `listingprojection.Apply(changes)` 刷新相关 HPD projection。
8. 返回更新后的 room。

## 3. 数据读写

读取：

- `hs_hmd_room_centralized`
- `hs_hpd_root_scope_relation`

写入：

- `hs_hmd_room_centralized`
- 相关 HPD projection

## 4. 失败返回

- `id` 非法：`InvalidParam`
- 房间不存在：`NotFound`
- 当前房东无权更新：`NotFound`
- HMD 更新失败：`DatabaseError`
- projection 刷新失败：内部错误 / projection error
