# 管理后台前端页面设计文档

本文档按页面定义 admin 前端第一阶段的路由、权限、接口、字段、交互和验收。

## 1. 通用页面规则

### 1.1 布局

受保护页面统一使用 admin layout：

- 左侧菜单：固定第一阶段模块，根据权限过滤。
- 顶部栏：当前员工姓名、手机号、角色摘要、退出按钮。
- 内容区：列表页以筛选栏、操作栏、表格、分页组成。

菜单顺序：

1. 房源上架审核
2. 房源管理
3. 发房方管理
4. 员工管理
5. 角色权限

`launch_audit` 未实现前，菜单不默认展示；页面文件可以先建，路由可暂不挂菜单。

### 1.2 表格

列表页通用能力：

- 筛选项变化不自动请求，点击“查询”后请求。
- “重置”清空筛选并回到第一页。
- 分页切换保留筛选条件。
- 操作成功后停留当前页刷新。
- 当前页因禁用等操作变空时，允许回退上一页后重查。

### 1.3 状态文案

通用状态：

| 值 | 文案 |
| --- | --- |
| `1` | 启用 / 正常 |
| `-1` | 禁用 |

房源状态、发布状态、审核状态进入实现时以 schema 状态字典为准，不在页面里散落硬编码。

### 1.4 错误展示

- 后端返回 `error` 时直接展示。
- 参数校验错误优先在表单项下展示，提交后端仍失败时展示全局 message。
- 401 清理 admin token 并跳转 `/login`。
- 403 展示无权限页面或模块内无权限空状态。

## 2. 登录页

### 2.1 路由

```text
/login
```

### 2.2 文件

```text
src/views/login/LoginPage.vue
src/api/adminAuth.ts
src/model/auth.ts
src/stores/adminAuth.ts
src/utils/auth/adminAuthToken.ts
```

### 2.3 接口

- `POST /api/v1/admin_auth/login`
- `POST /api/v1/admin_auth/session`

### 2.4 表单字段

| 字段 | 必填 | 校验 | 说明 |
| --- | --- | --- | --- |
| `phone` | 是 | 中国大陆手机号格式 | 员工手机号。 |
| `password` | 是 | 非空 | 员工密码。 |

### 2.5 交互

1. 已登录且 session 恢复成功时，访问 `/login` 自动跳到第一个有权限模块。
2. 点击登录时禁用按钮，防重复提交。
3. 登录成功后保存 admin token、员工资料、角色和权限。
4. 登录失败展示后端错误，例如 `手机号或密码错误，请重新输入`。
5. 不展示“忘记密码”入口；reset password 第一阶段不做。

### 2.6 验收

- 登录成功后能进入后台。
- 刷新页面后能恢复 session。
- 账号无业务权限时进入无权限空状态。
- 本仓库只验证 admin 登录态，不保留 publish token 兼容用例。

## 3. Admin Layout

### 3.1 文件

```text
src/layout/AdminLayout.vue
src/layout/AdminSidebar.vue
src/layout/AdminHeader.vue
src/stores/permission.ts
```

### 3.2 菜单配置

| 路由 | 名称 | 权限 |
| --- | --- | --- |
| `/launch-audits` | 房源上架审核 | 待定 |
| `/houses` | 房源管理 | `house.view` |
| `/providers` | 发房方管理 | `provider.view` |
| `/staff` | 员工管理 | `staff.view` |
| `/roles` | 角色权限 | `role.view` |

### 3.3 顶部栏

展示：

- 员工姓名。
- 手机号。
- 角色名摘要或角色编码摘要。
- 退出登录。

退出：

- 调用 `POST /api/v1/admin_auth/logout`。
- 成功后清理本地状态并跳转 `/login`。
- token 已失效时也清理本地状态。

## 4. 员工管理

### 4.1 路由

```text
/staff
```

权限：

- 访问页面：`staff.view`
- 新建 / 编辑 / 禁用：`staff.edit`

### 4.2 文件

```text
src/views/staff/StaffListPage.vue
src/views/staff/StaffDrawer.vue
src/api/staff.ts
src/model/staff.ts
```

### 4.3 接口

- `POST /api/v1/staff/list`
- `POST /api/v1/staff/detail`
- `POST /api/v1/staff/create`
- `POST /api/v1/staff/update`
- `POST /api/v1/staff/disable`
- `POST /api/v1/role/list`，用于角色下拉。

### 4.4 筛选区

| 控件 | 请求字段 | 说明 |
| --- | --- | --- |
| 关键词输入 | `keyword` | 匹配姓名等后端支持字段。 |
| 手机号输入 | `phone` | 精确或后端支持的手机号筛选。 |
| 角色下拉 | `role_id` | 从 role list 取选项。 |
| 状态下拉 | `status` | 不传默认有效员工；`-1` 查禁用员工。 |

### 4.5 表格列

| 列 | 字段 | 说明 |
| --- | --- | --- |
| 员工姓名 | `name` | 空显示 `-`。 |
| 手机号 | `phone` | 账号唯一标识。 |
| 角色 | `role_names` | 多个用逗号展示；空显示“无角色”。 |
| 部门 | `department` | 空显示 `-`。 |
| 岗位 | `job_title` | 空显示 `-`。 |
| 状态 | `status` | 标签。 |
| 创建时间 | `created_at` | 时间格式化。 |
| 最近登录 | `last_login_at` | 空显示 `-`。 |
| 最近登录 IP | `last_login_ip` | 空显示 `-`。 |
| 操作 | - | 详情、编辑、禁用。 |

### 4.6 详情抽屉

调用 `staff/detail`。

展示分区：

- 基础资料：姓名、手机号、邮箱、部门、岗位、联系二维码。
- 角色信息：角色名称、角色编码。
- 账号状态：状态、创建人、创建时间、更新时间。
- 登录信息：密码更新时间、最近登录时间、最近登录 IP。

### 4.7 新建员工抽屉

字段：

| 字段 | 必填 | 校验 |
| --- | --- | --- |
| `name` | 是 | 非空。 |
| `phone` | 是 | 手机号格式。 |
| `password` | 是 | 非空；长度规则以后端为准。 |
| `email` | 否 | 邮箱格式可前端弱校验。 |
| `department` | 否 | 文本。 |
| `job_title` | 否 | 文本。 |
| `contact_qr_code` | 否 | URL 文本。 |
| `role_ids` | 否 | 多选角色。 |

提交接口：`staff/create`。

### 4.8 编辑员工抽屉

字段：

- 可编辑：姓名、邮箱、部门、岗位、联系二维码、角色、状态。
- 不可编辑：手机号、密码。

提交接口：`staff/update`。

注意：

- `role_ids` 不传表示不调整角色，传空数组表示清空角色。
- 前端编辑表单提交时应明确传当前选择后的 `role_ids`，避免用户以为已保存但实际未变更。

### 4.9 禁用确认

点击禁用：

- 弹窗展示员工姓名和手机号。
- 二次确认后调用 `staff/disable`。
- 成功后刷新列表。

## 5. 角色权限

### 5.1 路由

```text
/roles
```

权限：

- 访问页面：`role.view`
- 新建 / 编辑：`role.edit`

### 5.2 文件

```text
src/views/role/RoleListPage.vue
src/views/role/RoleDrawer.vue
src/api/role.ts
src/model/role.ts
src/model/permission.ts
```

### 5.3 接口

- `POST /api/v1/role/list`
- `POST /api/v1/role/detail`
- `POST /api/v1/role/create`
- `POST /api/v1/role/update`

### 5.4 筛选区

| 控件 | 请求字段 | 说明 |
| --- | --- | --- |
| 关键词输入 | `keyword` | 搜索角色名称或角色编码。 |

### 5.5 表格列

| 列 | 字段 |
| --- | --- |
| 角色名称 | `role_name` |
| 角色编码 | `role_code` |
| 权限摘要 | `permission_names` |
| 系统内置 | `is_system` |
| 状态 | `status` |
| 创建时间 | `created_at` |
| 更新时间 | `updated_at` |
| 操作 | 详情、编辑 |

### 5.6 详情 / 编辑抽屉

详情调用 `role/detail`。

展示：

- 角色名称。
- 角色编码。
- 描述。
- 权限列表：权限名称、权限编码、模块、动作。
- 是否系统内置。
- 状态、创建时间、更新时间。

编辑字段：

| 字段 | 创建 | 编辑 | 说明 |
| --- | --- | --- | --- |
| `role_name` | 可填 | 可填 | 必填。 |
| `role_code` | 可填 | 只读 | 创建后不可修改。 |
| `description` | 可填 | 可填 | 可空。 |
| `permission_codes` | 必填 | 可调整 | 创建至少 1 个；编辑不能提交空数组。 |

权限选择：

- 第一版按 `module` 分组展示 checkbox。
- 如果后端没有独立权限字典接口，先用已知权限字典构建前端静态只读选项，并在文档中标记需要后端补 `permission/list`。
- 提交后端仍以权限码合法性校验为准。

## 6. 发房方管理

### 6.1 路由

```text
/providers
```

权限：

- 访问页面：`provider.view`
- 新建 / 编辑 / 禁用：`provider.edit`

### 6.2 文件

```text
src/views/provider/ProviderListPage.vue
src/views/provider/ProviderDrawer.vue
src/api/provider.ts
src/model/provider.ts
```

### 6.3 接口

- `POST /api/v1/provider/list`
- `POST /api/v1/provider/detail`
- `POST /api/v1/provider/create`
- `POST /api/v1/provider/update`
- `POST /api/v1/provider/disable`

### 6.4 筛选区

| 控件 | 请求字段 | 说明 |
| --- | --- | --- |
| 手机号输入 | `phone` | 发房方登录手机号。 |
| 状态下拉 | `status` | 正常 / 禁用。 |

### 6.5 表格列

| 列 | 字段 | 说明 |
| --- | --- | --- |
| 发房方 ID | `provider_id` | 可复制。 |
| 手机号 | `phone` | 当前阶段核心识别字段。 |
| 状态 | `status` | 标签。 |
| 创建人 | `created_by_staff_id` | 当前只有 ID。 |
| 更新人 | `updated_by_staff_id` | 当前只有 ID。 |
| 创建时间 | `created_at` | 时间格式化。 |
| 更新时间 | `updated_at` | 时间格式化。 |
| 密码更新时间 | `password_updated_at` | 空显示 `-`。 |
| 最近登录 | `last_login_at` | 空显示 `-`。 |
| 最近登录 IP | `last_login_ip` | 空显示 `-`。 |
| 操作 | - | 详情、编辑手机号、禁用。 |

不展示：

- 城市。
- 发房方名称。
- 房源数量。
- 资质资料。
- 品牌资料。

### 6.6 新建发房方抽屉

字段：

| 字段 | 必填 | 校验 |
| --- | --- | --- |
| `phone` | 是 | 手机号格式。 |
| `password` | 是 | 非空；长度规则以后端为准。 |

提交接口：`provider/create`。

### 6.7 编辑发房方抽屉

字段：

| 字段 | 必填 | 校验 |
| --- | --- | --- |
| `provider_id` | 是 | 从当前记录带入，不允许编辑。 |
| `phone` | 是 | 手机号格式。 |

提交接口：`provider/update`。

### 6.8 禁用确认

- 弹窗展示 `provider_id` 和手机号。
- 二次确认后调用 `provider/disable`。
- 禁用成功后刷新列表和详情。

## 7. 房源管理

### 7.1 路由

```text
/houses
```

权限：

- 访问页面和详情：`house.view`

### 7.2 文件

```text
src/views/house/HouseListPage.vue
src/views/house/HouseDetailDrawer.vue
src/api/house.ts
src/model/house.ts
```

### 7.3 接口

- `POST /api/v1/house/list`
- `POST /api/v1/house/detail`

### 7.4 筛选区

| 控件 | 请求字段 | 说明 |
| --- | --- | --- |
| 发房方 ID | `provider_id` | 可从 provider 页面跳转带入。 |
| 资产类型 | `asset_mode` | 集中式 / 分散式，枚举以 schema 为准。 |
| 城市 | `city` | 文本或后续城市选择器。 |
| 区域 | `district` | 文本或后续区域选择器。 |
| 房态 | `room_status` | 枚举标签。 |
| 发布状态 | `listing_status` | 枚举标签。 |
| 审核状态 | `audit_status` | 枚举标签。 |

### 7.5 表格列

| 列 | 字段 |
| --- | --- |
| 房源标题 | `title` |
| 发房方 | `provider_phone` / `provider_id` |
| 资产类型 | `asset_mode` |
| 位置 | `city` + `district` + `biz_area` |
| 项目 / 小区 | `building_name` / `community_name` |
| 房间号 | `room_no` |
| 租金 | `price_text`，兜底 `price` |
| 户型 | `layout_text` |
| 面积 | `area_size` |
| 房态 | `room_status` |
| 发布状态 | `listing_status` |
| 审核状态 | `audit_status` |
| 在线 | `is_online` |
| 更新时间 | `updated_at` |
| 操作 | 详情 |

### 7.6 详情抽屉

调用 `house/detail`。

展示分区：

1. 房源摘要：标题、资产类型、租金、户型、面积、房间号。
2. 位置归属：城市、区域、商圈、地址、项目、小区、楼栋、房型。
3. 发房方：`provider_id`、`provider_phone`、`landlord_name`。
4. 状态：房态、发布状态、审核状态、是否在线。
5. 审核摘要：最新审核任务 ID、提交时间、审核时间、审核人。
6. 系统信息：source type、source ID、root type、root ID、创建时间、更新时间。

限制：

- 不提供编辑按钮。
- 不提供上架、下架、审核按钮。
- 不提供导出按钮。

## 8. 房源上架审核

### 8.1 路由

```text
/launch-audits
/launch-audits/:audit_id
```

状态：

- 页面设计保留。
- 后端实现和权限点未完成前不进入可用菜单。

### 8.2 文件

```text
src/views/launch-audit/LaunchAuditListPage.vue
src/views/launch-audit/LaunchAuditDetailPage.vue
src/views/launch-audit/LaunchAuditDecisionDialog.vue
src/api/launchAudit.ts
src/model/launchAudit.ts
```

### 8.3 计划接口

- `POST /api/v1/launch_audit/list`
- `POST /api/v1/launch_audit/detail`
- `POST /api/v1/launch_audit/approve`
- `POST /api/v1/launch_audit/reject`

### 8.4 计划列表筛选

| 控件 | 请求字段 |
| --- | --- |
| 来源 | `source` |
| 发房方 | `provider_id` |
| 资产类型 | `asset_mode` |
| 状态 | `status` |

### 8.5 计划列表列

需要后端补齐字段后确认，前端页面至少需要：

- 审核单 ID。
- 房源标题或发布对象摘要。
- 发房方手机号。
- 资产类型。
- 审核状态。
- 提交时间。
- 审核时间。
- 审核人。
- 操作：详情。

### 8.6 计划详情页

详情页必须显示：

1. 审核单基础信息。
2. 当前生效内容。
3. 待审核内容。
4. 差异摘要。
5. 审核记录。
6. 审核备注输入区。

审核通过：

- 弹窗输入可选备注。
- 调用 `launch_audit/approve`。
- 成功后刷新详情，禁止重复提交。

审核驳回：

- 弹窗输入必填驳回原因。
- 调用 `launch_audit/reject`。
- 成功后刷新详情，禁止重复提交。

### 8.7 阻塞项

进入实现前必须补齐：

1. 审核权限点。
2. 审核单详情响应字段。
3. 当前生效内容、待审核内容、差异摘要的数据结构。
4. 审核状态枚举。
5. 已处理审核单重复处理时的错误码和中文提示。

## 9. 403 / 无权限页

### 9.1 路由

```text
/403
```

### 9.2 触发场景

- 登录成功但没有任何业务权限。
- 访问无权限路由。
- session 恢复后权限变化，当前页面不再可访问。

### 9.3 展示

- 标题：当前账号无权访问该页面。
- 辅助信息：请联系管理员调整角色权限。
- 操作：退出登录。

不展示权限码调试信息给普通用户；开发环境可以在 console 输出。
