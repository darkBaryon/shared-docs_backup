# POST /api/v1/decentralized_room/list_by_community

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/room.(*Handler).ListDecentralizedByCommunity`
- service：`internal/service/publish.(*decentralizedRoomService).ListDecentralizedRoomsByCommunity`

## 2. 处理逻辑

1. handler 绑定 `decentralized_id`。
2. service 构造 `PublishScope`。
3. 校验当前房东是否可访问 `community_id`。
4. 通过后按 `community_id` 查询 `hs_hmd_room_decentralized`。
5. 返回该小区下全部房间。
6. 当前实现里，无权访问父级时返回空数组 `[]`。

## 3. 数据读写

读取：

- `hs_hpd_root_scope_relation`
- `hs_hmd_room_decentralized`

写入：

- 无

## 4. 失败返回

- `decentralized_id` 非法：`InvalidParam`
- scope 查询失败：`DatabaseError`
- 房间查询失败：`DatabaseError`

## 5. 关键点

- 单父级查询，先 scope 再查 HMD
