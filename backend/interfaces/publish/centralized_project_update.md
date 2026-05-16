# POST /api/v1/centralized_project/update

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/project.(*Handler).Update`
- service：`internal/service/publish.(*centralizedProjectService).UpdateCentralizedProject`

## 2. 处理逻辑

1. handler 绑定更新字段。
2. service 先校验当前房东是否能访问该 `project_id`。
3. 权限通过后，调 `domain/hmd.UpdateCentralizedProject` 更新 HMD。
4. HMD 返回 change。
5. service 调 `listingprojection.Apply(changes)` 刷新相关 HPD 投影。
6. 返回更新后的 project。

## 3. 数据读写

读取：

- `hs_hpd_root_scope_relation`
- `hs_hmd_centralized`

写入：

- `hs_hmd_centralized`
- 受 change 影响的 HPD 投影表

## 4. 失败返回

- 当前房东无权更新：`NotFound`
- 项目不存在：`NotFound`
- HMD 更新失败：`DatabaseError`
- projection 刷新失败：内部错误 / projection error

