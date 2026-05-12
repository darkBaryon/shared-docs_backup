# 房源审核与变更模块

## 1. 模块目标

本模块用于管理房源从录入到发布过程中的审核、变更与状态流转。

该模块用于保证：

- 上架前有审核闭环
- 变更后有审计记录
- 发布动作可回溯

## 2. 集合清单

- `house_audit_task`
- `house_change_request`
- `house_change_audit`
- `house_publish_log`

## 3. 集合设计

### 3.1 `house_audit_task`

**功能说明**  
房源审核任务。用于管理房源发布前的审核流程。

**核心字段**

- `audit_task_id`：审核任务唯一标识
- `listing_id`：关联 `house_listing.listing_id`
- `audit_type`：审核类型，例如首次上架、重新上架
- `audit_status`：审核状态，例如待审核、通过、驳回
- `submitted_by`：提单人
- `reviewer_staff_id`：审核人
- `audit_remark`：审核备注
- `submitted_at`：提交时间
- `reviewed_at`：审核完成时间

### 3.2 `house_change_request`

**功能说明**  
房源变更申请。用于记录房源信息更新的申请内容。

**核心字段**

- `change_request_id`：变更申请唯一标识
- `listing_id`：关联 `house_listing.listing_id`
- `change_scope`：变更范围，例如价格、图片、标签、联系人
- `before_snapshot`：变更前快照
- `after_snapshot`：变更后快照
- `submitted_by`：提单人
- `request_status`：申请状态
- `submitted_at`：提交时间
- `updated_at`：更新时间

### 3.3 `house_change_audit`

**功能说明**  
房源变更审核记录。用于承接变更申请的审核结果与审批意见。

**核心字段**

- `change_audit_id`：变更审核唯一标识
- `change_request_id`：关联 `house_change_request.change_request_id`
- `audit_status`：审核状态
- `reviewer_staff_id`：审核人
- `audit_remark`：审核说明
- `reviewed_at`：审核时间

### 3.4 `house_publish_log`

**功能说明**  
房源发布日志。用于记录上架、下架、重新发布等发布动作。

**核心字段**

- `publish_log_id`：发布日志唯一标识
- `listing_id`：关联 `house_listing.listing_id`
- `action_type`：动作类型，例如上架、下架、恢复
- `operator_id`：操作人
- `operator_type`：操作人类型，例如员工、系统
- `action_remark`：操作说明
- `created_at`：操作时间
