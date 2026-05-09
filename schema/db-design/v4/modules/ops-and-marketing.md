# 运营与营销

集合：
- `hs_ops_banner`
- `hs_ops_popup`
- `hs_ops_featured_slot`
- `hs_ops_push_task`

字段要点：
- 广告/弹窗：`position_code` `image_url` `target_type` `effective_from/to`
- 优质位：`listing_ids` `city` `sort_order`，`listing_ids` 关联 `hs_hpd_listing._id`
- 推送任务：`target_scope` `delivery_channel` `task_status` `scheduled_at`
