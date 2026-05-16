# POST /api/v1/room_type/create

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/roomtype.(*Handler).Create`
- service：`internal/service/publish.(*roomTypeService).CreateRoomType`

## 2. 处理逻辑

1. handler 绑定 `project_id / building_id` 与房型基础字段。
2. service 构造 `PublishScope`。
3. 如果请求带 `building_id`，先校验 building 对应 project 是否可访问。
4. 否则按 `project_id` 校验 root scope。
5. 通过后调用 `domain/hmd.CreateRoomType` 写入 HMD。
6. HMD 返回 change。
7. service 调 `listingprojection.Apply(changes)` 刷新相关 HPD projection。
8. 返回创建后的 room type。

## 3. 数据读写

读取：

- `hs_hpd_root_scope_relation`
- `hs_hmd_building`（按 building 维度创建时）

写入：

- `hs_hmd_room_type_centralized`
- 相关 HPD projection

## 4. 失败返回

- 入参不合法：`InvalidParam`
- 当前房东无权访问父级：`NotFound`
- room type 唯一性或依赖校验失败：`AlreadyExists / InvalidParam`
- HMD 写入失败：`DatabaseError`
- projection 刷新失败：内部错误 / projection error

## 5. 关键约束

- room type 不单独存 owner
- room type 继承 project / building 对应的 root scope
- room type 的核心作用是“房间创建模板”，集中式房间在 `create` 时传入 `room_type_id` 后，未显式填写的公共字段会用 room type 模板补齐
