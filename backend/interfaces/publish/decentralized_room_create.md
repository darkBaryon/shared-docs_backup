# POST /api/v1/decentralized_room/create

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/room.(*Handler).CreateDecentralized`
- service：`internal/service/publish.(*decentralizedRoomService).CreateDecentralizedRoom`

## 2. 处理逻辑

1. handler 绑定 `decentralized_id` 与房间基础字段。
2. service 构造 `PublishScope`。
3. 校验当前房东是否可访问该 community。
4. 调 `domain/hmd.CreateDecentralizedRoom` 写入 `hs_hmd_room_decentralized`。
5. HMD 返回 `HmdMutationResult`。
6. service 调 `listingprojection.Apply(changes)` 刷新：
   - `hs_hpd_listing`
   - `hs_hpd_publisher_listing`
   - miniapp 相关 projection
7. 返回创建后的房间。

## 3. 数据读写

读取：

- `hs_hpd_root_scope_relation`

写入：

- `hs_hmd_room_decentralized`
- `hs_hpd_listing`
- `hs_hpd_publisher_listing`

## 4. 失败返回

- 入参不合法：`InvalidParam`
- 当前房东无权访问目标小区：`NotFound`
- HMD 写入失败：`DatabaseError`
- projection 刷新失败：内部错误 / projection error
