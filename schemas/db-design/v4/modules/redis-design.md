# Redis 设计

用途：会话、缓存、计数、分布式锁（不存主数据）。

键设计：
- `hs:sess:{token}` -> `user_id`（TTL=会话过期）
- `hs:cache:listing:detail:{listing_id}`（TTL=120s）
- `hs:cache:listing:list:{hash_query}`（TTL=60s）
- `hs:counter:staff:daily_push:{staff_id}:{yyyymmdd}`（TTL=3d）
- `hs:lock:index_rebuild:{listing_id}`（TTL=30s）

失效：
- 发布/下架/审核通过后，删 listing 相关缓存
- 收藏/足迹变化后，删用户聚合缓存
