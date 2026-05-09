# 用户行为

集合：
- `hs_usr_favorite`
- `hs_usr_history`
- `hs_usr_plan`
- `hs_usr_notification`

字段要点：
- 收藏：`user_id + listing_id` 唯一，`listing_id` 关联 `hs_hpd_listing._id`
- 足迹：`viewed_at`，建议 TTL 90 天
- 看房计划：`listing_ids` + `plan_status`，`listing_ids` 关联 `hs_hpd_listing._id`
- 通知：`is_read` + `related_id`
