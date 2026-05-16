# POST /api/v1/building/create

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/building.(*Handler).Create`
- service：`internal/service/publish.(*buildingService).CreateBuilding`

## 2. 处理逻辑

1. handler 绑定 building 基础字段与 `project_id`。
2. service 从 context 读取 publish principal，构造 `PublishScope`。
3. 先校验当前房东是否可访问目标 `project_id`。
4. scope 通过后，调用 `domain/hmd.CreateBuilding` 写入 `hs_hmd_building`。
5. HMD 返回 `HmdMutationResult`。
6. service 调 `listingprojection.Apply(changes)` 刷新受影响的 read model。
7. 返回创建后的 building。

## 3. 数据读写

读取：

- `hs_hpd_root_scope_relation`
- `hs_hmd_centralized`

写入：

- `hs_hmd_building`
- 受 change 影响的 HPD projection

## 4. 失败返回

- `project_id` 非法或入参绑定失败：`InvalidParam`
- 当前房东无权访问该项目：`NotFound`
- project 不存在或父级关系不合法：`NotFound / InvalidParam`
- building 编码/唯一性冲突：`AlreadyExists`
- HMD 写入失败：`DatabaseError`
- projection 刷新失败：内部错误 / projection error

## 5. 关键约束

- building 继承 project root scope，本体不写 owner 字段。
- 不允许向非本人项目下创建楼栋。
