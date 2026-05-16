# POST /api/v1/centralized_project/list

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/project.(*Handler).List`
- service：`internal/service/publish.(*centralizedProjectService).ListCentralizedProjects`

## 2. 当前查询主线

这条接口已经改成：

1. 先从 `hs_hpd_root_scope_relation` 查当前房东可见的 `project_ids`
2. 再带着这些 ids 去 HMD 查 `hs_hmd_centralized`
3. 同时叠加 `city / district` 业务筛选

不再走“先查一批 HMD 项目，再在应用层过滤”的旧路径。

## 3. 数据读写

读取：

- `hs_hpd_root_scope_relation`
- `hs_hmd_centralized`

写入：

- 无

## 4. 返回逻辑

- 当前房东无任何 root scope：直接返回空列表
- 当前房东有 root scope，但业务筛选后无结果：返回空列表

## 5. 失败返回

- scope 查询失败：`DatabaseError`
- HMD 查询失败：`DatabaseError`

## 6. 关键约束

- 这条接口的第一收口条件必须是 owner scope，不是 city/district。
- city/district 只是缩小当前房东已有范围，不负责权限判断。

