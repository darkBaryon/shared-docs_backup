# Dashboard Schema

## Tenant Dashboard

- `favorite_count`：收藏数量。
- `history_count`：浏览历史数量。
- `session_count`：会话数量。

## Landlord Dashboard

- `house_total`：房源总数。
- `house_on_rent`：待租数（status=1）。
- `house_rented`：已租数（status=2）。
- `house_offline`：下架数（status=0）。

## 统计口径

- 统计字段用于个人主页展示。
- 推荐实时聚合，避免将统计值冗余写入用户主表。
