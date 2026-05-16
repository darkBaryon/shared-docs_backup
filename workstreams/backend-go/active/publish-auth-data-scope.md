# Publish Auth 与数据作用域计划

状态：按“房东登录、房东只看自己的主数据树”主线执行；root owner scope relation 与 create 链补偿一致性已落地，待清理剩余旧口径。

## 结论

- `publish web` 是房东端，不是员工端。
- `publish_auth/login` 的 `phone` 是房东手机号，不是员工手机号。
- `publish` session principal 固定为：
  - `terminal=publish`
  - `principal_type=landlord`
- publish 数据作用域主线应改为：
  - `project -> owner`
  - `community -> owner`
  - `building / room_type / room` 继承父级 owner scope
- 不做兼容层，不保留 staff 与 user 双主线并存的 publish 方案。

## 已确认边界

- HMD 不新增 owner/user/staff 字段。
- 前端不传 `user_id`、`staff_id`、`owner_phone` 之类身份过滤字段。
- publish 业务页面结构、自动加载逻辑、token/session 持久化与恢复机制保留。
- publish 主线不再依赖 `hs_adm_staff`、`adm_role`、`adm_permission`、`super_admin`、`house.manage`。
- 后台员工全量查看属于后台系统职责，不复用 publish 房东端 scope。

## 后端主线

### Publish Auth

- 保留接口：
  - `POST /api/v1/publish_auth/login`
  - `POST /api/v1/publish_auth/session`
  - `POST /api/v1/publish_auth/logout`
- `login` 正式使用房东账号密码登录。
- 登录主体来自 `hs_lld_landlord.phone`，认证数据来自 `hs_lld_auth.password_hash`。
- session principal 改为 `principal_type=landlord`。
- `role_codes`、`permission_codes` 保留字段，但当前对 publish 主线为空数组。

### Publish Scope

- 访问范围由后端从 publish session principal 推导。
- root owner scope relation 已用于表达：
  - `centralized_project -> owner`
  - `decentralized_community -> owner`
- `building / room_type / room` 不单独存 owner，统一继承父级 scope：
  - 集中式：`project -> building -> room_type / room`
  - 分散式：`community -> room`
- 不存在 publish 全局 staff 放开路径。

### 数据库完善方向

- root owner scope relation collection 已定为 `hs_hpd_root_scope_relation`。
- 字段最小集合建议：
  - `root_type`：`centralized_project` / `decentralized_community`
  - `root_id`
  - `owner_landlord_id`
  - `owner_phone`（冗余快照，可选）
  - `relation_status`
  - `effective_from`
  - `effective_to`
- 只表达 root 主档归属，不混入 listing/service/staff 语义。
- `root_type + root_id` 表达一个 root 主档的唯一 active 归属；`owner_landlord_id` 是房东主体外键与查询键；`owner_phone` 不能作为长期归属主键，只能作为冗余快照。
- `POST /api/v1/centralized_project/create`、`POST /api/v1/decentralized_community/create` 必须保证 “HMD 主档 + HPD projections + root scope relation” 同成同败；当前 Mongo 未启用事务，按应用层补偿回滚实现。

### 为什么要补这一层

- 当前 schema 没有 `project/community` 级 owner relation。
- 房东先创建空项目/空小区，再继续创建楼栋/房型/房间，是刚性主流程。
- 如果没有 root owner scope，房东创建完空主档后会立刻因 scope 丢失而看不到它，业务链断裂。

### 旧垃圾代码清理范围

- 删除 publish 主线中的 `staff` principal 分支。
- 删除 `super_admin` / `house.manage` / role / permission 对 publish scope 的影响。
- 删除旧的 listing 反推 `project/community` 可见性的主路径。
- 删除 publish auth / service / test / 文档中残留的员工端权限口径。

### 实施顺序

1. 先补 shared-docs schema / architecture / publish scope 设计稿。
2. 已新增 root owner scope model / repository / indexes。
3. 已接入 `create centralized project` / `create decentralized community` 自动写 root owner scope。
4. 已接入 `project/community` 的 list/detail/update 走 root owner scope。
5. 已接入 `building/room_type/room` 继承父级 scope。
6. 已补 `create centralized project` / `create decentralized community` 的应用层补偿回滚。
7. 最后清理剩余旧 `staff` 口径和测试垃圾代码。

## 文档口径

所有 publish 文档统一按以下表述维护：

- “房东手机号登录”
- “publish 登录主体是房东 landlord”
- “房东只能看自己名下的项目 / 小区 / 楼栋 / 房型 / 房间”
- “前端不传身份过滤字段”

旧员工端口径不得再作为 publish 主线出现；涉及后台员工、角色、权限的描述只能归入 admin 后台场景。

## 测试要求

- `publish_auth/login`：存在 `hs_lld_landlord.phone` 且 `hs_lld_auth.password_hash` 校验通过时可成功登录。
- `publish_auth/session`：只接受 `publish + landlord` session。
- root owner scope：创建项目 / 小区后，房东立刻可见且可继续新增下级对象。
- `publish service`：房东只能看和修改自己名下的项目、小区、楼栋、房型、房间。
