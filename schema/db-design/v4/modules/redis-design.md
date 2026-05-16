# Redis 设计

用途：会话、缓存、计数、分布式锁（不存主数据）。

键设计：
- `hs:sess:{token}` -> structured principal JSON（TTL=会话过期）
- `hs:cache:listing:detail:{listing_id}`（TTL=120s）
- `hs:cache:listing:list:{hash_query}`（TTL=60s）
- `hs:counter:staff:daily_push:{staff_id}:{yyyymmdd}`（TTL=3d）
- `hs:lock:index_rebuild:{listing_id}`（TTL=30s）

`hs:sess:{token}` 约束：

- Redis session 只保存登录态，不保存账号密码真值源；
- principal 至少包含 `principal_type`、`principal_id`、`terminal`、`phone`；
- `miniapp` 使用 `principal_type=user`；
- `publish` 使用 `principal_type=landlord`；
- `admin` 使用 `principal_type=staff`。

失效：
- 发布/下架/审核通过后，删 listing 相关缓存
- 收藏/足迹变化后，删用户聚合缓存
