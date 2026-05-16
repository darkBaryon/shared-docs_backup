# POST /api/v1/building/list_by_project

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/building.(*Handler).ListByProject`
- service：`internal/service/publish.(*buildingService).ListBuildingsByProject`

## 2. 处理逻辑

1. handler 绑定 `project_id`。
2. service 构造 `PublishScope`。
3. 先判断当前房东是否可访问这个 project。
4. 可访问时，按 `project_id` 查询 `hs_hmd_building`。
5. 返回该项目下全部 building。
6. 当前实现里，如果 project 不在当前房东 scope 内，直接返回空数组 `[]`。

## 3. 数据读写

读取：

- `hs_hpd_root_scope_relation`
- `hs_hmd_building`

写入：

- 无

## 4. 失败返回

- `project_id` 非法：`InvalidParam`
- scope 查询失败：`DatabaseError`
- building 查询失败：`DatabaseError`

## 5. 关键点

- 这条接口本来就是“先父级 scope，再子级查询”，没有“先查一批再过滤”的问题。
- 当前无权访问父级时返回 `[]`，不是 `404`。
