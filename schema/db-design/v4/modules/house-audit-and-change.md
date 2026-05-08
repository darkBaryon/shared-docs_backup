# 房源审核与变更（V4）

## 1. 范围

- 本文档定义房源审核与变更流程集合。
- 公共字段见 [common-fields.md](./common-fields.md)。
- 本模块统一使用缩写 `hac`（house audit change）。

## 2. 集合定义

### 2.1 `hs_hac_audit_task`

用途：房源上架审核任务。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `listing_id` | objectId | 是 | 无 | 关联 `hs_hpd_listing._id` |
| `audit_type` | string | 是 | `"initial_publish"` | 审核类型 |
| `audit_status` | int | 是 | `1` | 审核状态 |
| `submitted_by` | objectId | 否 | 无 | 提交人（员工） |
| `reviewer_staff_id` | objectId | 否 | 无 | 审核人 |
| `audit_remark` | string | 否 | `""` | 审核备注 |
| `submitted_at` | int64 | 是 | 当前时间秒 | 提交时间 |
| `reviewed_at` | int64 | 否 | `0` | 审核完成时间 |

索引：
- `listing_id_1_audit_status_1`
- `audit_status_1_submitted_at_-1`
- `reviewer_staff_id_1_audit_status_1`

### 2.2 `hs_hac_change_request`

用途：房源信息变更申请。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `listing_id` | objectId | 是 | 无 | 关联 `hs_hpd_listing._id` |
| `change_scope` | array<string> | 是 | `[]` | 变更范围字段列表 |
| `before_snapshot` | object | 是 | 无 | 变更前快照 |
| `after_snapshot` | object | 是 | 无 | 变更后快照 |
| `submitted_by` | objectId | 否 | 无 | 提交人（员工） |
| `request_status` | int | 是 | `1` | 申请状态 |
| `submitted_at` | int64 | 是 | 当前时间秒 | 提交时间 |

索引：
- `listing_id_1_request_status_1`
- `request_status_1_submitted_at_-1`
- `submitted_by_1_submitted_at_-1`

### 2.3 `hs_hac_change_audit`

用途：变更申请审核记录。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `change_request_id` | objectId | 是 | 无 | 关联 `hs_hac_change_request._id` |
| `audit_status` | int | 是 | `1` | 审核状态 |
| `reviewer_staff_id` | objectId | 否 | 无 | 审核人 |
| `audit_remark` | string | 否 | `""` | 审核说明 |
| `reviewed_at` | int64 | 否 | `0` | 审核时间 |

索引：
- `change_request_id_1`（唯一）
- `audit_status_1_reviewed_at_-1`

### 2.4 `hs_hac_publish_log`

用途：上架/下架/恢复等发布动作审计日志。

| 字段 | 类型 | 必填 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| `listing_id` | objectId | 是 | 无 | 关联 `hs_hpd_listing._id` |
| `action_type` | string | 是 | 无 | 动作类型 |
| `operator_id` | objectId | 否 | 无 | 操作人 |
| `operator_type` | string | 否 | `""` | 操作人类型（staff/system） |
| `action_remark` | string | 否 | `""` | 操作说明 |
| `action_at` | int64 | 是 | 当前时间秒 | 动作时间 |

索引：
- `listing_id_1_action_at_-1`
- `action_type_1_action_at_-1`
- `operator_id_1_action_at_-1`

## 3. 枚举字典

### 3.1 `audit_type`

| 值 | 含义 |
| --- | --- |
| `initial_publish` | 首次上架审核 |
| `republish` | 重新上架审核 |
| `info_change` | 信息变更审核 |

适用字段：
- `hs_hac_audit_task.audit_type`

### 3.2 `audit_status`

| 值 | 含义 |
| --- | --- |
| `0` | 未指定 |
| `1` | 待审核 |
| `2` | 审核通过 |
| `3` | 审核驳回 |
| `-1` | 已关闭 |

适用字段：
- `hs_hac_audit_task.audit_status`
- `hs_hac_change_audit.audit_status`

### 3.3 `request_status`

| 值 | 含义 |
| --- | --- |
| `0` | 未指定 |
| `1` | 待审核 |
| `2` | 审核通过 |
| `3` | 审核驳回 |
| `-1` | 已关闭 |

适用字段：
- `hs_hac_change_request.request_status`

### 3.4 `action_type`

| 值 | 含义 |
| --- | --- |
| `publish` | 上架 |
| `offline` | 下架 |
| `recover` | 恢复 |
| `rent_out` | 标记已出租 |
| `delete` | 逻辑删除 |

适用字段：
- `hs_hac_publish_log.action_type`

## 4. 流程关联约定

- `hs_hac_audit_task` 审核通过后，更新 `hs_hpd_listing.listing_status=3`。
- `hs_hac_audit_task` 审核驳回后，保持 `hs_hpd_listing.listing_status` 为非上架态，并写 `audit_remark`。
- `hs_hac_change_request` 审核通过后，将 `after_snapshot` 应用到 `hs_hpd_listing`，并刷新 `hs_hpd_listing_index`。
- 所有发布动作必须写 `hs_hac_publish_log` 审计记录。
