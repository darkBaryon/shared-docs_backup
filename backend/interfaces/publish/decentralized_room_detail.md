# POST /api/v1/decentralized_room/detail

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/room.(*Handler).DecentralizedDetail`
- service：`internal/service/publish.(*decentralizedRoomService).GetDecentralizedRoom`

## 2. 处理逻辑

1. handler 绑定 `id`。
2. service 构造 `PublishScope`。
3. 先读 `hs_hmd_room_decentralized`。
4. 用 `room.decentralized_id` 判断当前房东是否拥有对应 community。
5. 通过后返回，无权访问按 not found 处理。

## 3. 数据读写

读取：

- `hs_hmd_room_decentralized`
- `hs_hpd_root_scope_relation`

写入：

- 无

## 4. 失败返回

- `id` 非法：`InvalidParam`
- 房间不存在：`NotFound`
- 当前房东无权访问：`NotFound`
- 查询失败：`DatabaseError`
