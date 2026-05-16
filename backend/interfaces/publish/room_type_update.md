# POST /api/v1/room_type/update

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/roomtype.(*Handler).Update`
- service：`internal/service/publish.(*roomTypeService).UpdateRoomType`

## 2. 处理逻辑

1. handler 绑定房型更新字段。
2. service 构造 `PublishScope`。
3. 先读 room type。
4. 根据其 `project_id / building_id` 判断 scope。
5. 权限通过后更新 HMD。
6. HMD 返回 change。
7. service 调 `listingprojection.Apply(changes)` 刷新相关 HPD projection。
8. 返回更新后的 room type。

## 3. 数据读写

读取：

- `hs_hmd_room_type_centralized`
- `hs_hmd_building`（必要时）
- `hs_hpd_root_scope_relation`

写入：

- `hs_hmd_room_type_centralized`
- 相关 HPD projection

## 4. 失败返回

- `id` 非法：`InvalidParam`
- room type 不存在：`NotFound`
- 当前房东无权更新：`NotFound`
- HMD 更新失败：`DatabaseError`
- projection 刷新失败：内部错误 / projection error
