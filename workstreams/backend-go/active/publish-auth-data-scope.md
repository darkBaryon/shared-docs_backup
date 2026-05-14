# Publish Auth 与数据作用域计划

状态：阶段 1 后端 publish auth 已实现并 review 通过，待阶段 2 前端登录态接入。

## 背景

当前出房 Web 已能录入集中式项目、楼栋、房型、房间以及分散式小区、房间，但 publish 路由只是复用了通用 Redis session 中间件。只要 token 有效，就能看到 HMD 全量项目/小区；这适合联调，不符合真实业务。

这次目标不是给 HMD 增加 `owner` 字段。按 V4 数据库设计，HMD 只保存房源主数据结构；用户、员工、权限在账号域；房源归属和服务关系在 HPD 的 `hs_hpd_entrust_relation`。

## 已确认事实

- `hs_hmd_centralized`、`hs_hmd_building`、`hs_hmd_decentralized`、`hs_hmd_room_*` 不包含 owner/user/staff 字段，也不应新增。
- `hs_adm_staff`、`hs_adm_role`、`hs_adm_permission`、`hs_adm_staff_role`、`hs_adm_role_permission` 已在 schema 中定义，但 Go 代码尚未建 model/repository/service。
- `hs_hpd_listing` 已由 HPD projector 根据 HMD 房间生成，`source_type` 当前只支持 `centralized_room` / `decentralized_room`。
- `hs_hpd_entrust_relation` 已在 schema 中定义，用于 `listing_id` 与 `owner_phone`、`maintainer_staff_id`、`service_staff_id` 的关系，但 Go 代码尚未实现。
- 旧 Python 的 `house.owner_id` 是旧写模型做法，迁移到 V4 Go 后不能直接复制到 HMD。
- `backend/auth.md` 已明确：小程序 auth 不作为 publish/admin 的通用登录入口，publish/admin 应建立自己的 `service/{terminal}/auth`。

## 目标

1. 建立 publish Web 自己的 auth/session 接口，让前端不再靠手工粘贴 token。
2. 后端根据 session principal 自动判断数据作用域，前端不传 `user_id`、`staff_id` 或 owner 过滤条件。
3. 项目/小区列表默认加载当前 principal 可维护的数据；搜索框只做筛选，不决定权限。
4. 不改 HMD schema，不新增 HMD owner 字段，不把小程序 auth 当 publish auth。
5. 按阶段开发、按阶段 review；每阶段通过后再进入下一阶段。

## 数据作用域规则

### Principal

session 里需要保存可识别终端和身份的 principal，而不是只有字符串 `user_id`：

```text
principal_type: user | staff
principal_id: ObjectID hex
terminal: miniapp | publish | admin
phone: string
role_codes: string[]
permission_codes: string[]
```

小程序接口使用 `principal_type=user`。publish Web MVP 优先使用 `principal_type=staff`，对应 `hs_adm_staff`；后续房东端登录如复用 publish Web，可用 `principal_type=user` 并通过 `hs_usr_user.phone -> hs_hpd_entrust_relation.owner_phone` 取作用域。

### 全局权限

满足任一条件可读写全部 HMD/HPD 发房数据：

- `role_codes` 包含 `super_admin`
- `permission_codes` 包含 `house.manage`

其他 staff 只读写自己服务或维护的房源：

```text
hs_hpd_entrust_relation.relation_status = 1
and (
  maintainer_staff_id = current_staff_id
  or service_staff_id = current_staff_id
)
```

user 类型 principal 只读写自己电话对应的房源：

```text
hs_hpd_entrust_relation.relation_status = 1
and owner_phone = current_user.phone
```

### HMD 聚合规则

权限从 listing 起算，再聚合回 HMD：

- 集中式房间：`hs_hpd_listing.source_type=centralized_room` 且 `source_id=hs_hmd_room_centralized._id`。
- 分散式房间：`hs_hpd_listing.source_type=decentralized_room` 且 `source_id=hs_hmd_room_decentralized._id`。
- 集中式项目列表：当前 principal 可维护的集中式房间所属 `project_id` 集合。
- 分散式小区列表：当前 principal 可维护的分散式房间所属 `decentralized_id` 集合。
- 项目/小区搜索：先套权限作用域，再按 city/district/name 等查询条件筛选。

注意：按当前 schema，空项目、空楼栋、空小区在还没有房间/listing 前没有可表达的个人归属。MVP 规则是：全局权限账号可维护空 HMD 主档；非全局账号的“我的项目/小区”由已有 listing 的 entrust relation 聚合得出。不要为了解决空主档归属而给 HMD 加 owner。

## 阶段 0：契约确认

后端 agent：

- 更新 `api/publish.md`，新增 publish auth 接口契约：
  - `POST /api/v1/publish_auth/login`
  - `POST /api/v1/publish_auth/session`
  - `POST /api/v1/publish_auth/logout`
- 更新 `backend/auth.md`，说明 session principal JSON 结构和 miniapp/publish 的边界。
- 更新 `backend/publish.md`，说明 publish service 必须接收 principal 并执行数据作用域校验。

前端 agent：

- 更新 `frontend/publish.md`，说明正式流程使用 publish auth，开发 token 面板只允许 `VITE_ENABLE_DEV_TOKEN_PANEL=true` 时出现。
- 明确前端永远不提交 `user_id` / `staff_id` / owner 过滤字段。

Review 停点：

- review agent 只审文档契约是否闭环，尤其确认 HMD 不新增归属字段。

## 阶段 1：后端 publish auth

后端 agent：

- 增加账号域 Go model/repository：
  - `hs_adm_staff`
  - `hs_adm_role`
  - `hs_adm_permission`
  - `hs_adm_staff_role`
  - `hs_adm_role_permission`
  - `hs_adm_login_log`
- 改造 `pkg/session`，让 Redis session 存 principal JSON；小程序 auth 和 publish auth 都通过同一个 session store 颁发 opaque token。
- 新增 `internal/service/publish/auth` 和 `handler/v1/publish/auth`。
- `publish_auth/login` MVP 按员工手机号登录：
  - `env=local` 可只传 `phone`，用于本地联调。
  - 非 local 环境必须预留验证码/外部认证入口，不在没有 schema 的情况下硬塞 password 字段。
- `publish_auth/session` 返回当前 principal 的 staff/user 摘要、角色和权限。
- publish 业务路由改挂 publish 专用 middleware：必须是 `terminal=publish` 且 principal 有发房访问资格。

建议测试：

```text
go test ./pkg/session ./internal/handler/v1/publish ./internal/service/publish/...
```

Review 停点：

- review agent 检查 session 是否没有继续假设纯 `userId` 字符串。
- 检查小程序 auth 测试是否同步调整并仍通过。
- 检查 publish auth 没有复用小程序微信 login。

## 阶段 2：前端登录态接入

前端 agent：

- 新增 `src/api/publishAuth.ts`、`src/model/publishAuth.ts`、`src/stores/auth.ts`。
- 新增登录页或登录面板，调用 `publish_auth/login` 获取 token。
- App 初始化时调用 `publish_auth/session` 恢复登录态。
- Axios 继续只放 `Authorization: Bearer <token>`，不解析 token。
- 401 或业务码 `10002` 时清理 token 并回到登录态。
- 项目/小区列表在 session 有效后自动加载；查询按钮只用于重新筛选。
- 开发 token 面板保留在环境开关后面，不作为主流程。

建议验证：

```text
npm run lint
npm run build
```

Review 停点：

- review agent 检查登录后是否自动进入并加载项目/小区。
- 检查前端没有把 `user_id`、`staff_id`、owner 字段塞进业务请求。

## 阶段 3：后端 HPD entrust relation

后端 agent：

- 增加 HPD model/repository：
  - `HpdEntrustRelation`
  - `HpdContact` 如当前链路需要联系人查询可一起补，否则先不接 UI。
- repository 至少提供：
  - `UpsertActiveByListingID`
  - `FindActiveByListingID`
  - `ListActiveListingIDsByStaff`
  - `ListActiveListingIDsByOwnerPhone`
  - `CanAccessListing`
- HPD domain 暴露根据 source 查 listing 的窄方法，供 publish service 在房间创建后拿到 listing_id。
- 房间创建成功并完成 HPD projector 后，publish service 自动创建/更新 entrust relation：
  - staff principal：默认 `maintainer_staff_id=current_staff_id`，`service_staff_id=current_staff_id`。
  - user principal：默认 `owner_phone=current_user.phone`。
  - 不把这些字段回写到 HMD。
- 给 relation 补索引初始化或迁移脚本，索引名按 schema。

建议测试：

```text
go test ./internal/model ./internal/repository/hpd ./internal/domain/hpd ./internal/service/publish
```

Review 停点：

- review agent 检查 HMD model/repository 没有新增 owner/staff 字段。
- 检查 room create 的 HMD 写入、HPD listing、entrust relation 是同一业务动作内可追踪的链路。

## 阶段 4：后端数据作用域

后端 agent：

- 在 publish service 层引入 `PublishScope`，所有 publish 读写都从 context principal 获取 scope。
- 列表接口：
  - 全局权限：保持现有 HMD 查询，并支持 city/district 筛选。
  - 非全局 staff/user：先通过 entrust relation 找 listing，再回查 HMD 上层项目/小区。
- 详情、更新、房态接口：
  - 全局权限直接允许。
  - 非全局 principal 必须能通过 listing relation 证明访问权；否则返回 `10005` 或权限错误码，不能泄露目标存在性。
- 子对象列表：
  - 项目/小区不可访问时返回空列表或资源不存在。
  - 房间列表必须只返回当前 principal 可维护的房间。
- 创建接口：
  - 全局权限可创建项目、楼栋、小区、房型、房间。
  - 非全局账号创建房间时必须生成自己的 entrust relation。
  - 非全局账号创建空项目/空小区后，是否长期在“我的列表”出现按上方 HMD 聚合规则处理，不用 HMD owner 兜底。

建议测试：

```text
go test ./internal/handler/v1/publish ./internal/service/publish ./internal/domain/hpd
```

需要覆盖：

- A 员工只能看到 A relation 下的项目/小区/房间。
- B 员工不能 detail/update A 的房间。
- `super_admin` 或 `house.manage` 可见全量。
- city/district 是筛选，不是必填权限条件。

Review 停点：

- review agent 用双账号 fixture 查跨账号泄露。
- 检查所有 publish 入口都走 service 层 scope，不在 handler 或前端绕过。

## 阶段 5：前端作用域体验

前端 agent：

- 登录后自动加载项目/小区，空态文案表达“暂无可维护数据”。
- 搜索条件只传 city/district/name 等业务筛选，不传身份筛选。
- 工作台内楼栋、房型、房间 tab 的列表刷新以接口返回为准，不做本地权限推断。
- 当前登录人信息显示在布局区域，方便联调确认账号。

建议验证：

```text
npm run lint
npm run build
```

Review 停点：

- review agent 使用两个不同员工 token 在浏览器中验证列表隔离。
- 检查登录态过期后不会停留在假登录页面。

## 联调验收

本地服务：

```text
backend: http://127.0.0.1:8080
frontend: http://127.0.0.1:5173
```

验收用例：

1. 员工 A 登录，创建集中式房间，后端生成 HMD room、HPD listing、entrust relation。
2. 员工 A 刷新 `/centralized/projects`，自动看到该房间所属项目。
3. 员工 B 登录，看不到员工 A 的项目/房间，直接访问详情也失败。
4. 管理员登录，可看到全部项目/小区。
5. 搜索城市只缩小当前账号可见范围，不会扩大权限。

## Agent 协作规则

- 一个阶段一个开发 agent 实施，一个 review agent 立即 review；不要等全量做完才 review。
- 每阶段 review 报告写到 `shared_docs/workstreams/{backend-go|publish-web}/reviews/`。
- review 通过后再进入下一阶段。
- 任一阶段发现需要改 schema，必须先回到阶段 0 更新契约；本计划默认不改 HMD schema。
