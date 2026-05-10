# 用户行为

集合：
- `hs_usr_favorite`
- `hs_usr_history`
- `hs_usr_plan`
- `hs_usr_notification`

字段要点：
- 收藏：`user_id + listing_id` 唯一，`listing_id` 关联 `hs_hpd_listing._id`
- 足迹：`user_id + listing_id` 唯一，重复浏览更新 `viewed_at` 和来源；`viewed_at` 建议 TTL 90 天
- 看房计划：`listing_ids` + `plan_status`，`listing_ids` 关联 `hs_hpd_listing._id`
- 通知：`is_read` + `related_id`

初始化：
- `hs_usr_favorite.user_id_1_listing_id_1` 和 `hs_usr_history.user_id_1_listing_id_1` 由 Go 服务启动时初始化。
