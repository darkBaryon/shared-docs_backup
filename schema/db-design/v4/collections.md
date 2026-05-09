# V4 集合与职责（跨端统一）

## A. 账号与权限（usr/adm）
- `hs_usr_user`：C 端用户主档
- `hs_usr_auth`：C 端认证绑定
- `hs_usr_profile_ext`：C 端偏好扩展
- `hs_adm_staff`：后台员工
- `hs_adm_role`：后台角色
- `hs_adm_permission`：后台权限点
- `hs_adm_staff_role`：员工-角色关系
- `hs_adm_role_permission`：角色-权限关系
- `hs_adm_login_log`：后台登录审计

## B. 房源主数据（hmd）
- `hs_hmd_centralized`：集中式主档
- `hs_hmd_building`：楼栋主档
- `hs_hmd_decentralized`：分散式主档
- `hs_hmd_room_type_centralized`：集中式房型模板
- `hs_hmd_room_centralized`：集中式房间实例
- `hs_hmd_room_decentralized`：分散式房间实例

## C. 发布与展示（hpd）
- `hs_hpd_listing`：发布主实体 / 统一 listing identity
- `hs_hpd_miniapp_listing`：小程序检索与展示 read model
- `hs_hpd_contact`：对外联系信息（后续接入）
- `hs_hpd_entrust_relation`：委托关系与服务归属（后续接入）

后续按端独立设计：

- `hs_hpd_admin_listing`：后台管理 read model，当前不建
- `hs_hpd_publisher_listing`：发房端/房东端 read model，当前不建

## D. 审核与变更（hac）
- `hs_hac_audit_task`：上架审核任务
- `hs_hac_change_request`：变更申请
- `hs_hac_change_audit`：变更审核记录
- `hs_hac_publish_log`：发布动作审计

## E. 用户行为（usr）
- `hs_usr_favorite`：收藏
- `hs_usr_history`：足迹
- `hs_usr_plan`：看房计划
- `hs_usr_notification`：站内通知

## F. 客户跟进（lead）
- `hs_lead_customer`：客户主档
- `hs_lead_requirement`：需求快照
- `hs_lead_appointment`：预约看房
- `hs_lead_followup`：跟进记录
- `hs_lead_match`：匹配记录

## G. 运营营销（ops）
- `hs_ops_banner`：广告位
- `hs_ops_popup`：弹窗
- `hs_ops_featured_slot`：推荐位
- `hs_ops_push_task`：推送任务
