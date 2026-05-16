# POST /api/v1/building/update

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/building.(*Handler).Update`
- service：`internal/service/publish.(*buildingService).UpdateBuilding`

## 2. 处理逻辑

1. handler 绑定 building 更新字段。
2. service 构造 `PublishScope`。
3. 先读 `hs_hmd_building`，拿到 `building.project_id`。
4. 校验当前房东对该 project 的访问权。
5. 通过后调用 `domain/hmd.UpdateBuilding` 更新 HMD。
6. HMD 返回 change。
7. service 调 `listingprojection.Apply(changes)` 刷新受影响的 HPD projection。
8. 返回更新后的 building。

## 3. 数据读写

读取：

- `hs_hmd_building`
- `hs_hpd_root_scope_relation`

写入：

- `hs_hmd_building`
- 相关 HPD projection

## 4. 失败返回

- `id` 非法：`InvalidParam`
- building 不存在：`NotFound`
- 当前房东无权更新：`NotFound`
- HMD 更新失败：`DatabaseError`
- projection 刷新失败：内部错误 / projection error
