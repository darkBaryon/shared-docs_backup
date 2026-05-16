# POST /api/v1/building/detail

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/building.(*Handler).Detail`
- service：`internal/service/publish.(*buildingService).GetBuilding`

## 2. 处理逻辑

1. handler 绑定 `id`。
2. service 构造 `PublishScope`。
3. 先从 HMD 读取 `hs_hmd_building`。
4. 如果 building 存在，取出 `building.project_id`。
5. 通过 project root scope 判断当前房东是否可访问。
6. 通过后返回 building；无权访问按 not found 处理。

## 3. 数据读写

读取：

- `hs_hmd_building`
- `hs_hpd_root_scope_relation`

写入：

- 无

## 4. 失败返回

- `id` 非法：`InvalidParam`
- building 不存在：`NotFound`
- 当前房东无权访问该 building 所属项目：`NotFound`
- 查询失败：`DatabaseError`
