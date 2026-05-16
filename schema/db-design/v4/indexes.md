# V4 索引策略（核心）

## 1. 唯一约束
- `hs_usr_user`: `phone_1`
- `hs_usr_auth`: `auth_provider_1_openid_1`
- `hs_usr_profile_ext`: `user_id_1`
- `hs_lld_landlord`: `phone_1`
- `hs_lld_auth`: `landlord_id_1`
- `hs_lld_profile`: `landlord_id_1`
- `hs_adm_staff`: `phone_1`
- `hs_adm_staff_auth`: `staff_id_1`
- `hs_adm_role`: `role_code_1`
- `hs_adm_permission`: `permission_code_1`
- `hs_adm_staff_role`: `staff_id_1_role_id_1`
- `hs_adm_role_permission`: `role_id_1_permission_id_1`
- `hs_hpd_listing`: `source_type_1_source_id_1`
- `hs_hpd_miniapp_listing`: `listing_id_1`
- `hs_usr_favorite`: `user_id_1_listing_id_1`
- `hs_usr_history`: `user_id_1_listing_id_1`

## 2. 检索核心
- `hs_hpd_miniapp_listing`:
  - `is_online_1_city_1_district_1_price_1`
  - `is_online_1_rent_mode_1_price_1`
  - `is_online_1_weight_score_-1_updated_at_-1`
- `hs_hpd_listing`:
  - `listing_status_1_updated_at_-1`
- `hs_hpd_publisher_listing`:
  - `owner_landlord_id_1_updated_at_-1`
  - `root_type_1_root_id_1_updated_at_-1`
  - `room_status_1_listing_status_1_updated_at_-1`
- `hs_usr_favorite`:
  - `user_id_1_status_1_updated_at_-1`
- `hs_usr_history`:
  - `user_id_1_viewed_at_-1`

## 3. 流程核心
- `hs_hac_audit_task`: `audit_status_1_submitted_at_-1`
- `hs_hac_change_request`: `request_status_1_submitted_at_-1`
- `hs_lead_appointment`: `assigned_staff_id_1_appointment_status_1_appointment_time_-1`

## 4. 生命周期
- `hs_usr_history`: `viewed_at_1`（TTL，90天）
