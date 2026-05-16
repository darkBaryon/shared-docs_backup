# POST /api/v1/centralized_room/create

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/room.(*Handler).CreateCentralized`
- service：`internal/service/publish.(*centralizedRoomService).CreateCentralizedRoom`

## 2. 处理逻辑

1. handler 绑定 `project_id / building_id / room_type_id` 与房间基础字段。
2. service 构造 `PublishScope`。
3. 先校验当前房东是否可访问 `building_id` 所属 project。
4. 如果请求带 `room_type_id`，先读 room type，并校验它属于同一项目或同一楼栋。
5. 对房间输入做模板注入：
   - `layout_text`
   - `area_size`
   - `orientation`
   - `decoration_level`
   - `payment_cycle`
   - `rent / deposit / service_fee`
   - `agency_fee_mode / agency_fee_value`
   - `images`
   - `room_facilities`
6. 若 `layout_text` 为空，会优先按房型的 `room_count / hall_count / bathroom_count / kitchen_count` 组装，例如 `2室1厅1卫`。
7. 再调 `domain/hmd.CreateCentralizedRoom` 写入 `hs_hmd_room_centralized`。
8. HMD 返回 `HmdMutationResult`。
9. service 调 `listingprojection.Apply(changes)` 刷新：
   - `hs_hpd_listing`
   - `hs_hpd_publisher_listing`
   - miniapp 相关 projection
10. 返回创建后的房间。

## 3. 数据读写

读取：

- `hs_hmd_building`
- `hs_hmd_room_type_centralized`（传入 `room_type_id` 时）
- `hs_hpd_root_scope_relation`

写入：

- `hs_hmd_room_centralized`
- `hs_hpd_listing`
- `hs_hpd_publisher_listing`

## 4. 失败返回

- 入参不合法：`InvalidParam`
- 当前房东无权访问目标 building/project：`NotFound`
- 房型不属于当前项目/楼栋、父级依赖不合法、room_no 冲突等：`InvalidParam / AlreadyExists`
- HMD 写入失败：`DatabaseError`
- projection 刷新失败：内部错误 / projection error
