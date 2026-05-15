# Phase 4 Publish Scope Review

日期：2026-05-15

结论：不建议通过。Phase 4 的主线已经落在 service 层，`PublishScope`、HPD entrust relation 聚合、房间 detail/update/status 校验方向正确；但当前实现仍有几个会影响验收的数据作用域缺口，需要修正后再进入 Phase 5。

## Findings

### P1 分散式小区列表仍强制 city，违反 Phase 4 筛选条件约定

- 位置：
  - `internal/service/publish/decentralized_community.go:46`
  - `internal/domain/hmd/hmd_decentralized_community.go:50`
- 问题：
  Phase 4 明确要求 `city/district` 是筛选条件，不是权限条件；但 publish service 仍先调用 HMD `ListDecentralizedCommunities`，HMD 层在 `city == ""` 时直接返回 `InvalidParam`。这会导致非全局账号无法“不带城市地加载我的小区”，也会让前端登录后自动加载小区的 Phase 5 体验卡住。
- 建议：
  让 HMD 分散式小区列表支持空 city/district；或者在 publish service 对非全局 scope 先通过 accessible listings 聚合出 community IDs，再按可选 city/district 过滤。最终行为要保证不传 city 也能返回当前 principal 可维护的小区。

### P1 父级创建路径只读取 scope，没有校验父资源访问权

- 位置：
  - `internal/service/publish/building.go:23`
  - `internal/service/publish/room_type.go:25`
  - `internal/service/publish/centralized_room.go:21`
  - `internal/service/publish/decentralized_room.go:21`
- 问题：
  这些 create 方法只是 `newPublishScope`，随后直接写 HMD。非全局 staff 如果拿到别人的 `project_id` / `building_id` / `decentralized_id`，可以在对方项目、小区下创建楼栋、房型或房间。房间创建后还会登记自己的 entrust relation，进一步让这个 staff 通过聚合看到被注入的父级项目/小区。
- 建议：
  对有父资源的 create 加父级 scope 校验：创建楼栋/房型/房间前必须证明当前 principal 可访问对应项目、楼栋或小区；如果产品确认非全局只能创建房间，也应显式禁止非全局创建楼栋/房型。空项目/空小区创建需要单独明确规则，但不能让已有父资源被任意写入。

### P1 有任一房间 relation 即可更新共享父级主档，权限过宽

- 位置：
  - `internal/service/publish/centralized_project.go:58`
  - `internal/service/publish/building.go:63`
  - `internal/service/publish/room_type.go:80`
  - `internal/service/publish/decentralized_community.go:58`
- 问题：
  当前父级 update 通过“该项目/小区下有一个当前 principal 可维护的房间”来授权。这会让维护某一间房的 staff 修改项目、小区、楼栋、房型等共享主档，影响同一父级下其他 staff 的房源展示和投影数据。
- 建议：
  父级主档 mutation 默认收紧为全局权限；若业务确实需要非全局维护父级信息，应在 HPD/账号域补明确的父级维护关系或权限码，而不是用任一房间 relation 代表整棵 HMD 父级写权限。

### P2 service ports 有隐式依赖和未使用接口，文件架构不够清晰

- 位置：
  - `internal/service/publish/building.go:18`
  - `internal/service/publish/room_type.go:19`
  - `internal/service/publish/ports.go:40`
- 问题：
  `buildingService` / `roomTypeService` 通过 type assertion 从 HMD service 里偷偷拿 `GetCentralizedRoom` / `GetBuilding` 能力；同时 `roomTypeScopeDomain` 已定义但没有被构造函数使用。这会隐藏授权所需依赖，未来替换 fake 或拆 domain 时容易变成“编译通过、运行返回 not found”的误导行为。
- 建议：
  把 scope 依赖做成显式接口，例如 `buildingScopeDomain`、`roomTypeScopeDomain`，构造函数直接接收这些接口，删除静默 type assertion。

### P2 测试覆盖没有覆盖关键绕过路径

- 位置：
  - `internal/service/publish/service_test.go:283`
  - `internal/service/publish/service_test.go:375`
- 问题：
  现有测试覆盖了项目/小区/房间列表过滤和 centralized room update 拦截，但没有覆盖：
  - 分散式小区不传 city 的列表行为。
  - 非全局账号创建楼栋、房型、房间时父级不可访问的场景。
  - decentralized room detail/update/status 的跨账号拒绝。
  - 父级 update 是否应该只允许全局权限。
- 建议：
  在修复上述 P1 后补双账号 fixture 测试，尤其要证明 A 员工不能向 B 的项目/小区写入子资源，B 也不能 detail/update/status A 的分散式房间。

## Architecture Notes

- `PublishScope` 放在 service 层是正确方向，handler 没有参与权限拼接，符合 Phase 4 的分层目标。
- HPD entrust relation 没有回写 HMD，HMD model/repository 未发现新增 owner/staff 字段，符合 V4 约束。
- `scope_hmd.go` 当前通过 listing -> room -> project/community 做聚合，语义清楚，但实现是 N+1 查询。MVP 可接受；数据量上来后建议补批量读取或 repository 聚合方法，避免每次列表重复读取所有 accessible listings 和逐房间回查 HMD。

## Verification

已运行：

```bash
go test ./...
```

结果：通过。注意：当前测试通过不代表 Phase 4 验收通过，因为上述 P1 场景没有被测试覆盖。
