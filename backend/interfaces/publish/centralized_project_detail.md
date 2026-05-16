# POST /api/v1/centralized_project/detail

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/project.(*Handler).Detail`
- service：`internal/service/publish.(*centralizedProjectService).GetCentralizedProject`

## 2. 处理逻辑

1. handler 绑定 `id`。
2. service 读取当前 publish principal。
3. 调 `domain/hmd.GetCentralizedProject` 从 `hs_hmd_centralized` 读取项目。
4. 调 `publishaccess.CanAccessProjectForPrincipal` 判断当前房东是否拥有该 root。
5. 有权限则返回，无权限按 not found 处理。

## 3. 数据读写

读取：

- `hs_hmd_centralized`
- `hs_hpd_root_scope_relation`

写入：

- 无

## 4. 失败返回

- `id` 非法：`InvalidParam`
- 项目不存在：`NotFound`
- 当前房东不拥有该项目：`NotFound`
- 查询失败：`DatabaseError`

