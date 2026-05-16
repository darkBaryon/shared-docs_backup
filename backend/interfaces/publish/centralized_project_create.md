# POST /api/v1/centralized_project/create

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/project.(*Handler).Create`
- service：`internal/service/publish.(*centralizedProjectService).CreateCentralizedProject`

## 2. 处理逻辑

1. handler 绑定项目基础字段。
2. service 从 context 读取 publish principal。
3. 调 `domain/hmd.CreateCentralizedProject` 写入 HMD 主档 `hs_hmd_centralized`。
4. HMD 返回 `HmdMutationResult{Entity, Changes}`。
5. service 调 `domain/listingprojection.Service.Apply(changes)` 刷新 HPD 投影。
6. service 调 `domain/publishaccess.Service.UpsertRootScopeForPrincipal` 写入 `hs_hpd_root_scope_relation`：
   - `root_type=centralized_project`
   - `root_id=project_id`
   - `owner_landlord_id=current principal`
7. 返回创建后的 project。

## 3. 数据读写

写入：

- `hs_hmd_centralized`
- `hs_hpd_listing` / `hs_hpd_publisher_listing`（如果 change 涉及已有房源投影刷新）
- `hs_hpd_root_scope_relation`

## 4. 补偿逻辑

当前 Mongo 未启用事务，因此 create 走应用层补偿：

1. HMD create 成功
2. Apply(changes) 失败 -> 软删除刚创建的 HMD project
3. root scope 写入失败 -> 软删除刚创建的 HMD project

目标：不能留下“创建成功但当前房东自己看不到”的孤儿项目。

## 5. 失败返回

- 入参不合法：`InvalidParam`
- HMD 写入失败：`DatabaseError`
- HPD projection 失败：内部错误 / projection error
- root scope 写入失败：内部错误 / database error

## 6. 关键约束

- 空项目创建后，当前房东必须立刻可见。
- owner 不写入 HMD project 本体，只写 root scope relation。
